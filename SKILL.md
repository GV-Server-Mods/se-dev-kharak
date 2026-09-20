---
name: se-dev-kharak
description: >-
  Use this skill when developing, configuring, or balancing features for the GV: Deserts of Kharak (GVK)
  Space Engineers server. Authoritative guide covering Kharak server lore, factions, dynamic alliances,
  zone architecture (Z0-Z3), grid classes & point caps, siegable shields & siege drives, KOTH encounters,
  economy & scrap refining, ModAPI C# engineering standards (60 TPS, Havok/Clang mitigation, Torch PatchManager),
  SBC/ModAdjuster file conventions, and documentation budgets.
license: MIT
allowed-tools: Read
---

# SE Dev Kharak — GV: Deserts of Kharak (GVK) Engineering Guide

**Applies to Space Engineers version 1 on the GV: Deserts of Kharak (GVK) dedicated server.**

Engineering handbook, server context, ModAPI/Torch standards, and SBC/ModAdjuster conventions for GVK.

---

## Reference Links & Sources of Truth

- **Steam Server Rules & Gameplay Guide**: https://steamcommunity.com/sharedfiles/filedetails/?id=2781522559
- **Steam Workshop Mod Collection**: https://steamcommunity.com/sharedfiles/filedetails/?id=2650582206
- **Known solutions to common crashes or errors**: https://spaceengineers.wiki.gg/wiki/Modding/Reference/Known_Solutions_to_crashes_or_errors
- **Other useful SE modding references**: https://spaceengineers.wiki.gg/wiki/Modding/Reference
- **SBC CubeBlock Definition notes**: https://spaceengineers.wiki.gg/wiki/Modding/Reference/SBC/CubeBlocks/CubeBlock_Definition

---

## Core Engineering & Design Mandates

1. **Automated Enforcement (Zero "Trust Me Bro™" Rules)**: Always enforce server rules programmatically via code/ModAPI (auto-disabling illegal blocks, distance-based state overrides, damage filters). Never rely on written honor systems or manual admin inspection. If a rule exists, write code that makes violating it mechanically impossible.
2. **Sim-Speed & GC Optimization (Target: 60 TPS)**: Zero allocations in hot paths (`UpdateBeforeSimulation`), stepped 100-tick intervals for non-urgent checks, squared distances.
3. **Multiplayer Integrity**: Server-authoritative state (`IsServer`) with clean client synchronization.
4. **Defensive Coding**: Treat the SE engine with defensive rigor against Keen engine regressions, subgrid splits, entity lifecycle traps, and Havok solver saturation.
5. **Synchronized Steam Guide & Rules Mirroring**: Whenever changing or creating server rules, zone behaviors, block limits, or rebalances in code, notify the user so changes are mirrored into the Steam Guide and player documentation.
6. **Rule Simplicity & Low Cognitive Overhead**: Keep rules intuitive with minimal edge-case exceptions.
7. **Anti-Mod-Bloat Strictness**: Aggressively minimize adding new mod dependencies. Always prefer writing clean, lightweight C# scripts or XML overrides within `GVK_Settings` over pulling in heavy third-party mods.

---

## Quick Reference & Key Coordinates

- **Crossroads Tower (Origin)**: `GPS:Crossroads Tower:62495:28019:37195:`
- **Zones (from Crossroads Tower)**:
  - **Zone 0 (0 – 20 km)**: Safe Starter Hub. Strict PvE, production/weapons disabled, non-siegable shields.
  - **Zone 1 (20 – 35 km)**: Fledgling PvP & Salvage. Small derelicts, weapons/drills enabled, large production disabled.
  - **Zone 2 (35 – 50 km)**: Contested Desert. Medium derelicts, large-scale production unlocked.
  - **Zone 3 (> 50 km)**: Deep Desert / Gaalsien Heart. Heavy military wrecks, Gaalsien convoys, full uncapped warfare.
- **Shield Generators**: 250m radius, 1W power, non-siegable in Z0, siegable in Z1–Z3 (30-min drain, 15-min recharge, requires 100 Siege Chips).
- **KOTH Sites**: Khar Toba (Z3, all grids), Kalash Site (Z3, small grids), Crashed Starship (Z2, small rovers). 3km thruster and large grid shutoffs.

---

## Modular Reference Library (Progressive Disclosure)

Read the specific reference document needed for your current task:

| Reference Document | Key Topics Covered |
| :--- | :--- |
| **[references/server-context.md](./references/server-context.md)** | Factions, dynamic alliances, zone details, grid classes (UPs/MPs), siege mechanics, KOTH encounters, cleanup cadence, economy (CUs/RUs/scrap), custom logistics (MnM, pipelines, static drills, Grid Defender, dynamic beacons). |
| **[references/modapi-standards.md](./references/modapi-standards.md)** | Physics/Havok/Clang mitigation, 60 TPS sim-speed, multiplayer sync, Torch PatchManager vs Harmony, zero empty catch rule, SE type system, DamageSystem hot-path, memory leaks, PCU authorship, bitmap fonts, async disk I/O, Torch RPC. |
| **[references/sbc-conventions.md](./references/sbc-conventions.md)** | SBC naming (`CubeBlocks_*.sbc` vs `GVK_*.sbc`), single `<SubtypeId>` rule, ModAdjuster folder layout & rules, DefExtAPI vs ModAdjuster, C# XML doc standards, Comment & Documentation Budget. |

