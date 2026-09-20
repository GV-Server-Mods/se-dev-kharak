# Space Engineers ModAPI C# & Torch Plugin Standards

## 1. Physics, Havok & Voxel-Clang Mitigation (High Server Priority)
- **Voxel Phasing & Solver Saturation**: High-speed rover impacts against voxels can tunnel rigid bodies past the surface, trapping them in continuous Havok collision solver loops (cratering server sim-speed to 0.2 and summoning Clang).
- **Anti-Clang Engineering Standards**:
  - Proactively architect systems to detect voxel penetration / trapped grids (e.g. high angular velocity/physics jitter while stationary against voxels).
  - Implement automated unstuck / nudge routines: dampening/zeroing linear and angular velocities and nudging trapped grids along the local gravity up-vector.
  - Keep subgrid constraint counts and excessive suspension stiffness bounded to avoid physics feedback loops against terrain meshes.

---

## 2. Performance & Sim-Speed (Target: 60 TPS Server Health)
- **Zero Allocations in Hot Paths**:
  - Never allocate objects (`new List`, LINQ queries, lambdas/closures, boxing) in `UpdateBeforeSimulation()` or `UpdateAfterSimulation()`.
  - Prefer cached arrays or reusable `List<T>` buffers with `.Clear()`.
- **Stepped Updates**:
  - Use `UpdateBeforeSimulation100()` (`MyEntityUpdateEnum.EACH_100TH_FRAME`) for periodic distance/zone checks and boundary scans.
  - Use `UpdateBeforeSimulation10()` only for responsive gameplay mechanics.
- **Math Optimization**:
  - Always use `Vector3D.DistanceSquared(a, b) < radiusSquared` instead of `Vector3D.Distance()`.
- **Thread Safety & Static Collections**:
  - Use dedicated lock objects (`private static readonly object beaconLock = new object();`) when managing shared static manager lists accessed across blocks or events.

---

## 3. Multiplayer Synchronization & Architecture
- **Server Authority**:
  - Enforce critical state changes (enabling/disabling blocks, inventory modifications, damage overrides) only when `MyAPIGateway.Multiplayer.IsServer` is `true`.
  - When syncing visual effects or UI to clients, use `MyAPIGateway.Multiplayer.SendMessageToOthers` or ModAPI network handlers.
- **Event Lifecycle & Memory Leak Prevention**:
  - Wire up entity events in `UpdateOnceBeforeFrame()` or `Init()`.
  - Always deregister events in `OnRemovedFromScene()` or `Close()`.
  - Check `if (Entity == null || Entity.MarkedForClose) return;` at the start of component logic.
  - Safely unregister all static handlers and clean up static caches in session `UnloadData()`.

---

## 4. C# ModAPI & Torch Plugin Engineering Standards

### Patching Engines: Torch PatchManager Is NOT Harmony
- **GVK Torch plugins patch via `Torch.Managers.PatchManager`** (`PatchContext`/`GetPattern` + `Prefixes`/`Suffixes`/`Transpilers`), which is Torch's own MSIL rewriter (`DecoratedMethod`) - not HarmonyLib. The API is Harmony-*shaped* (a prefix returning `false` skips the original, etc.); the machinery is not Harmony. Commit works by writing a raw jump into the JIT'd method prologue (`AssemblyMemory.WriteJump`), with byte-level revert on unload.
- **Modern Torch installs ship no `0Harmony.dll`** and `Torch.dll` contains no HarmonyLib. Never add a `Lib.Harmony`/`0Harmony` reference to a Torch plugin, and never assume Harmony tooling (`Harmony.GetPatchInfo`, `AccessTools`, `[HarmonyPatch]` attributes, Harmony owner IDs) exists or can see Torch patches - it cannot. Torch detours live in `PatchManager.GetPattern(method)`'s rewrite pattern, not in any Harmony registry.
- **Raw-Harmony plugins are a different ecosystem**: Pulsar client plugins (PluginHub) and Magnetar/Quasar server plugins (MagnetarHub) - the `se-dev-plugin` skill ecosystem, e.g. `viktor-ferenczi/se-performance-improvements` - ship and use their own HarmonyLib. Same patch concepts, separate registries, separate IL rewriters.
- **Both engines on one method = hard conflict, never "compatible"**: two independent IL rewriters writing jumps over the same method prologue is undefined behavior. Conflict audits must check BOTH registries: (1) Torch rewrite patterns for detours from foreign assemblies, (2) HarmonyLib patch ownership via reflection ONLY when a Harmony runtime is actually loaded in the process.
- **Feature gap, not superiority**: Torch PatchManager covers `__instance`/`__result`/`__field_<name>`/bool-prefix/transpilers, but has no `__state`, no finalizers/`__exception`, no reverse patches, no attribute auto-discovery. Plugins needing those - or one codebase across Torch + Magnetar/Pulsar - ship raw Harmony. GVK plugins don't need any of it; if a future feature does, isolate it in one class with a unique Harmony ID, never target methods we patch via PatchManager, and extend the `PatchConflictAudit`.
- **Verify, don't recite**: claims about what wraps what and who ships what are checked against the actual binaries/source (scan `Torch.dll` for `HarmonyLib` strings, read TorchAPI source), not community lore. Legacy "Torch wraps Harmony" facts are obsolete.

### 1. Exception Handling & The Zero Empty Catch Rule
- **Empty `catch { }` is strictly banned**: Empty catch blocks swallow critical engine failures, mask Keen interface regressions, and make debugging impossible.
- **The Catch Cost**: Catching an exception in .NET triggers stack unwinding, metadata token lookups, and stack frame allocations costing several milliseconds of CPU time. In tick updates or damage handlers, caught exceptions will instantly collapse 60 TPS server sim-speed.
- **Defensive Null Guards First**: Always prefer direct precondition checks over `try/catch`:
  ```csharp
  if (grid == null || grid.Closed || grid.MarkedForClose) return;
  ```
- **Delegate Unhooking Does Not Throw**: C# delegate subtraction (`-=`) is null-safe and thread-safe. Never wrap event unhooking in `try/catch`.
- **Log Exceptions With Context**: If an exception must be caught (e.g. disk I/O or network serialization), always catch the specific exception type and log it with context at `Warn` or `Error`. Never use bare `catch { }`.

### 2. Space Engineers Type System & Polymorphism
- **Beware the Deep Inheritance Chain**: In Keen's engine, block hierarchy is deep:
  `MyEntity` → `MyCubeBlock` → `MyTerminalBlock` → `MyFunctionalBlock` → `MyBeacon` (as well as doors, reactors, batteries, etc.).
  Never assume a functional block is not a terminal block (`slim.FatBlock is MyTerminalBlock` is `true` for all functional blocks).
- **Program to ModAPI Interfaces (`IMy...`)**:
  Always prefer interfaces (`IMyBeacon`, `IMyTerminalBlock`, `IMyPowerProducer`) over concrete classes (`MyBeacon`). Interfaces guarantee seamless support for modded blocks, DLC variants, and framework expansions without hardcoded subtype IDs.

### 3. Hot-Path Optimization (DamageSystem & Ticks)
- **DamageSystem Target Realities**: `RaiseAfterDamageApplied` / `RegisterAfterDamageHandler` fires thousands of times per second during combat (every grinder tick, deformation pulse, and fragment impact).
  - In Keen's engine, grid damage events **exclusively pass concrete `MySlimBlock` targets**. Fat blocks (`MyCubeBlock`) and `MyCubeGrid` instances are never passed to this event.
  - Never allocate objects, run LINQ, format strings, or execute `try/catch` inside damage handlers. Collapse dispatch logic to:
    ```csharp
    if (info.Amount <= 0f) return;
    var slim = target as MySlimBlock;
    if (slim?.CubeGrid == null) return;
    LastDamageTimes[slim.CubeGrid.EntityId] = DateTime.UtcNow;
    ```
- **Zero-Allocation Enum Equality**: Never stringify enums (`relation.ToString().Equals(...)`). Calling `.ToString()` on an enum allocates a heap string and burns GC cycles. Always compare enums directly (`relation == MyRelationsBetweenPlayers.Enemies`).

### 4. Memory Leak Prevention & Static Collection Lifecycle
- **Entity Eviction on World Removal**: In long-running servers, static dictionaries tracking grid IDs (e.g. cooldowns, damage timestamps) grow monotonically and leak memory indefinitely. Always hook `MyEntities.OnEntityRemove` in `Init()` to evict entities when they close or despawn, and unhook in `Cleanup()`.
- **Cold-Path Pruning**: Sweep expired cache entries only on cold paths (e.g. admin commands, player interactions, or stepped 1000-tick intervals), never in hot simulation frames or damage loops.

### 5. PCU Authorship Semantics
- **Grid PCU vs. Player Limits**:
  - `grid.BlocksPCU` represents the total PCU of **all** blocks on that physical grid regardless of author. Use it strictly for **physical grid size caps**.
  - `grid.TransferBlocksBuiltByID(senderId, ...)` only transfers and refunds blocks built by that specific player (`slim.BuiltBy == senderId`).
  - Always calculate player refunds by summing `slim.PCU` specifically where `slim.BuiltBy == senderIdentityId` (accounting for functional vs construction stage PCU) to prevent multi-builder or captured derelict telemetry skew.

### 6. Client Presentation, UI & Internationalization
- **Bitmap Font Limits (Tofu Boxes)**: In-game chat and HUD toasts render using Keen's custom bitmap font textures, not system TrueType fonts. Non-ASCII characters like bullets (`•` / `U+2022`) or em-dashes (`—`) lack glyphs in SE fonts and render as blank missing-character "tofu" boxes. Always use plain ASCII characters (`- `, `*`, `->`, `[!]`) in client-facing text.
- **Culture-Invariant Numeric Parsing**: Servers run on diverse Windows system locales (e.g. `de-DE`, `fr-FR`) where decimals use commas (`3000,5`) instead of periods (`3000.5`). Always specify `CultureInfo.InvariantCulture` for all `float.Parse`, `int.Parse`, and `long.Parse` calls in commands and config parsers.

### 7. Simulation Sim-Speed & Asynchronous Disk I/O
- **Ban Synchronous Disk I/O on Game Thread**: Never call `File.AppendAllText`, `File.WriteAllText`, or blocking file streams on the game simulation thread. Disk contention or anti-virus scanning will cause sim-speed hitches.
- **Thread Pool Offloading**: Offload dedicated audit logs and file writes to `ThreadPool.QueueUserWorkItem` with a dedicated lock object to ensure sequential writes without stalling the simulation.

### 8. Torch RPC Interception & Fallback Etiquette
- **Never Swallow Player Clicks on Missing State**: When writing Torch prefix patches that intercept Keen network RPCs (e.g. `RemoveBlocksBuiltByID`), always return `true` (allow vanilla execution) if sender identity resolution fails (`senderIdentityId == 0L`). Returning `false` silently drops player input with zero feedback and no server logs.

