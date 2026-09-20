# se-dev-kharak: Space Engineers GVK Server Developer Skill & Reference Suite

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Space Engineers](https://img.shields.io/badge/Space%20Engineers-v1-blue.svg)](https://www.spaceengineersgame.com/)
[![Server: GV - Deserts of Kharak](https://img.shields.io/badge/Server-GV%3A%20Deserts%20of%20Kharak-red.svg)](https://steamcommunity.com/sharedfiles/filedetails/?id=2781522559)

A specialized developer skill and engineering reference suite for modding, balancing, and configuring the **GV: Deserts of Kharak (GVK)** Space Engineers dedicated server.

Equips AI coding assistants (Antigravity, Claude Code, Cursor, Cline) and server developers with server-specific architecture, zone rules, grid limits, ModAPI C# standards, Torch PatchManager guidelines, and SBC/ModAdjuster conventions.

---

## Overview & Background

Created by **Mike Dude** for the **GV: Deserts of Kharak (GVK)** rover-PvPvE server and developed in close pair-programming collaboration with **Google DeepMind's Gemini (Antigravity)**.

The skill partitions deep server lore, gameplay systems, and low-level engine engineering rules into an on-demand, token-optimized format using **progressive disclosure**:
- The main `SKILL.md` provides an immediate operational mental model and cheat sheet.
- Heavy reference manuals in `references/` are read on-demand only when relevant to the task at hand.

---

## Architecture & Reference Documents

| File | Scope |
| :--- | :--- |
| **[`SKILL.md`](./SKILL.md)** | Core engineering mandates (Automated enforcement, 60 TPS sim-speed, multiplayer integrity), reference links, and quick-reference coordinate/zone cheatsheets. |
| **[`references/server-context.md`](./references/server-context.md)** | Server lore, factions (`COALITION`, `GAALSIEN`, etc.), dynamic alliances, zone architecture (Zone 0 to 3), grid classes & UP/MP point caps, siegable shields & siege drives, KOTH encounters, cleanup cadence, economy (CUs/RUs/scrap), and custom logistics. |
| **[`references/modapi-standards.md`](./references/modapi-standards.md)** | Physics/Havok/Clang mitigation, 60 TPS sim-speed mandates, multiplayer synchronization, Torch PatchManager vs HarmonyLib, Zero Empty Catch rule, SE type system & polymorphism, DamageSystem hot-path realities, memory leak prevention, PCU authorship semantics, and async disk I/O. |
| **[`references/sbc-conventions.md`](./references/sbc-conventions.md)** | SBC file layout (`CubeBlocks_*.sbc` vs `GVK_*.sbc`), single `<SubtypeId>` rule, ModAdjuster folder structure & naming rules, DefExtAPI distinction, C# XML documentation standards, and the Comment & Documentation Budget. |

---

## Installation

### Antigravity / Gemini CLI
To install globally across all workspaces:
```powershell
Copy-Item -Recurse ".\se-dev-kharak" "$env:USERPROFILE\.gemini\config\skills\se-dev-kharak"
```

Or clone directly into your global skills directory:
```powershell
git clone https://github.com/GV-Server-Mods/se-dev-kharak.git "$env:USERPROFILE\.gemini\config\skills\se-dev-kharak"
```

### Claude Code / Cline
Add the skill path or import the directory into your agent's custom skills / prompts folder.

---

## License

MIT License — see [LICENSE](./LICENSE) for details.

