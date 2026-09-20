# SBC & ModAdjuster Conventions and Documentation Budget

## 1. SBC File Layout Convention

### Naming
- `CubeBlocks_*.sbc` — Keen-named files. **Pure vanilla tweaks only.** Safe to diff against Keen's `Content\Data\CubeBlocks\` originals.
- `GVK_*.sbc` — GVK-original block definitions, plus any vanilla tweaks that are functionally inseparable (e.g., all beacons in one file).

### Folder Structure
```
Content\Data\
├── Cubeblocks\          (CubeBlocks — case-insensitive on NTFS, git-tracked as Cubeblocks)
│   ├── CubeBlocks_*.sbc   — vanilla tweaks
│   ├── GVK_*.sbc          — custom blocks
│   └── ...
├── Factions\
├── Game\
├── ModAdjuster\
├── Particles\
├── Prefabs\
└── *.sbc files at root    — flat, matching Keen's vanilla layout
```

### Design Notes
- **Why `CubeBlocks_` for vanilla tweaks**: Keen's own files use `CubeBlocks_<Category>.sbc`. Keeping that naming means you can diff your override against Keen's file of the same name during an update. If a file is named `CubeBlocks_Energy.sbc`, you know it only contains overrides of vanilla reactors/engines, never custom SubtypeIds.
- **Why `GVK_` for custom blocks**: Files prefixed `GVK_` are GVK's own blocks. No vanilla SubtypeId lives in a `GVK_` file. This makes it immediately obvious which files are safe to diff against Keen's originals and which are yours to maintain.

### Merge Rule
- Never mix vanilla tweaks and custom blocks in the same file. If a functional group genuinely needs both (e.g., beacons), it goes in `GVK_` to signal that it's not a pure vanilla override.

---

## 2. SBC Syntax & XML Deserialization Quirks

### Strict Single `<SubtypeId>` per `<Id>` Block
- In Space Engineers SBC definitions (`MyObjectBuilder_...`), an `<Id>` block must contain strictly one `<TypeId>` and one `<SubtypeId>`.
- Stacking multiple `<SubtypeId>` elements within a single `<Id>` block causes Keen's XML deserializer to silently discard all subsequent `<SubtypeId>` entries, registering only the first.
- **Symptom**: Secondary definitions, actions, triggers, or components fail to load at runtime with missing profile errors (e.g., MES reports `Could Not Load Action Profile From Trigger: : [SubtypeId]`).
- **Rule**: Every distinct definition, component, trigger, or action MUST be encapsulated within its own standalone `<EntityComponent xsi:type="...">` (or `<Definition xsi:type="...">`) block with its own dedicated `<Id>`. Never combine multiple `<SubtypeId>` tags inside a single `<Id>` block.

---

## 3. ModAdjuster File Layout Convention

ModAdjuster files (`.xml`) follow the same Keen-mirroring structure as SBC files, with subfolders matching Keen's vanilla `Content\Data\` layout.

### Folder Structure
```
Content\Data\ModAdjuster\
├── ModAdjusterFiles.txt
├── CubeBlocks\                    (matches Content\Data\Cubeblocks\)
│   ├── CubeBlocks_*.xml           (vanilla block tweaks)
│   └── CubeBlocks_*.xml           (mod block adjustments - name the source mod)
├── Game\                          (matches Content\Data\Game\)
│   └── SessionComponents_*.xml
└── *.xml at root                  (flat, matching Keen's vanilla layout)
    ├── Components_*.xml
    ├── PhysicalItems_*.xml
    ├── BlockVariantGroups_*.xml
    └── VoxelMaterials_*.xml
```

### Naming
- **Subfolder paths in `ModAdjusterFiles.txt`**: Use backslash separators (e.g., `CubeBlocks\CubeBlocks_Thrusters.xml`). ModAdjuster resolves these via string concatenation: `"Data\\ModAdjuster\\" + name`.
- **File naming**: Mirror the SBC convention — `CubeBlocks_*.xml` for block adjustments, `Components_*.xml` for components, etc. When adjusting another mod's blocks, name the file after the source mod (e.g., `CubeBlocks_MESNpcThrusters.xml` for MES NPC thruster tweaks).
- **Vanilla vs mod adjustments**: Files adjusting vanilla definitions stay at root or in the appropriate subfolder. Files adjusting third-party mods get a suffix naming the source mod (e.g., `Components_ZoneChips.xml` for Kamikaze siege-drive economy, not `Components_Economy.xml`).

### Key Differences from SBC Files
- **Extension**: ModAdjuster files use `.xml`, not `.sbc`.
- **List file**: All adjustment files must be listed in `ModAdjusterFiles.txt` with their relative paths from `ModAdjuster\`.
- **No GVK_ prefix**: ModAdjuster files don't use the `GVK_` prefix — they're always adjustments to existing definitions, never new blocks.
- **DefExtAPI files are NOT ModAdjuster files**: Files containing `<ModExtensions>` with `<ModComponents>` and `<GameLogicComponent>` entries are Definition Extensions API files, not ModAdjuster files. These attach gamelogic and custom properties to definitions via `DefinitionExtensions.txt` at the Data root, not through ModAdjuster. The `<ModExtensions>` content looks similar to ModAdjuster adjustments but goes through a completely different pipeline (Draygo's DefExtAPI mod, workshop 2756894170). Never move DefExtAPI files into the ModAdjuster folder — doing so causes ModAdjuster to attempt deserializing them as definition adjustments, which leads to NREs when the definitions resolve to unexpected types.

---

## 4. Other Mod Details & Cross-File Integrity

- When working with Space Engineers mods, always suggest when changes need to be made on related `.sbc` files for things like `BlockVariantGroups`, `BlockCategories`, `BlueprintClasses`, `Components`, `EntityComponents`, etc., so that block changes are fully implemented across all gameplay mechanics.

---

## 5. C# Code Documentation & IntelliSense Standards

- Always write XML-style documentation comments (`/// <summary>`, `/// <param>`, `/// <returns>`) on all classes, structs, enums, public/internal methods, constructors, and key properties across all C# projects (Torch plugins, game mods, and PB scripts) — but keep each doc within the length budget below. Short and complete beats long and ignored.
- Ensure XML summaries explain the practical purpose of the component or method, describe any parameter units (e.g. m/s, frames, meters), and note any Keen/engine quirks or workarounds being addressed — in as few lines as the budget allows.
- Keep internal/inline `//` comments focused on non-obvious algorithms, edge cases, and performance considerations, within the budget.

---

## 6. Comment & Documentation Budget (Anti-Bloat Rules)

Comment bloat compounds: every AI agent pass historically added prose on top of prior prose, pushing worked files past 30% comment lines. These rules cap that. Violations get trimmed on sight.

- **Comments explain "why" for non-obvious logic only.** Never narrate *what* the adjacent code does, and never narrate *history*. No "Previously this did X", no `// FIX:` story paragraphs, no bug-narrative essays, no dated changelog comments ("removed 2026-08"). Bug history belongs in git commit messages; design discussion belongs in README.md, not the .cs file.
- **XML doc budget**: `<summary>` max 3 lines for methods/properties/fields (1-2 preferred); max 6 lines for classes/enums. `<param>`/`<returns>`: one short line each, and only when the name doesn't already say it.
- **Inline comment budget**: single-line `//` comments preferred. Multi-line block comments or banner rulers (`// ----`) only for genuine multi-step algorithm warnings, and at most one per method.
- **Cross-system dependency comments are the one mandatory exception**: any code whose correctness depends on another system (MES spawn configs, other GVK scripts, plugin settings, SBC definitions) always gets a one-line inline comment, e.g. "Friendly NPC structure protection lives in GVK_Derelicts MES config - do not remove this allow." These are never trimmed away.
- **Redundancy ban**: if the comment restates the code line it sits on (`info.Amount = 0f; // block PvP`), delete it unless it adds units, a quirk explanation, or a dependency warning.
- **Agent behavior - the teeth**: When modifying existing code, agents must NOT expand, re-word, or add to existing comments unless the logic change makes them wrong. Trim opportunistically: any comment touched during an edit must comply with this budget. Treat the budget as a hard ceiling - if a summary needs more lines to be clear, the method probably needs splitting, not more prose.
- **Design rationale relocation**: design discussion longer than the budget that is still worth keeping goes into a "Design Notes" subsection for that script in the repo's README.md.

