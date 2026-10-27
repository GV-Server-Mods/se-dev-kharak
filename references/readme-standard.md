# README standard for GVK repos

Read this before writing or changing a README, or a design doc, in any GVK repo. A README is the front door: it points, it generates, and it says timeless things. It is never the source of truth for a rule, a number or a status.

## The test for every line
Ask where the fact lives. If code, config, an SBC file, a skill or the hub tracker already holds it, the README links to it or a script generates it. If it truly has to be written by hand and it can change, it is registered in the GVK Wiki's `tools/facts.json` so the status script re-checks it every session. Anything else is timeless (a purpose, a philosophy, a credit) and can stay as prose.

## Sections, and what keeps each one current
Use the sections that apply; keep this order. Content mods (SBC and MES data) use all of them; script mods use the first, second, fourth, fifth and last three.

| Section | What goes in it | Kept current by |
| --- | --- | --- |
| What this is | Two sentences; the Workshop item if there is one | Timeless |
| Philosophy | The few ideas the repo is built on, one line each, no numbers | Timeless |
| Requirements | Load order, the framework mods, the minimum MES or game build | A registered fact, or a link to the design doc that owns it |
| Operator cheat sheet | Sandbox variables, and the admin and debug commands you reach for | Variables are generated from the data files. Commands are checked against their source (MES `Sync/ChatMessage.cs`) or carry a "checked on <date>" note |
| Naming | The prefab and file naming scheme | Generated, or a link to the generated reference |
| For other modders | What is worth copying and the traps, each item linking to a real file | A link check confirms the file exists |
| Folder map | One line per top-level folder | Reviewed when a folder is added |
| Tools | One line per script in `tools/` and how to run it | Reviewed when a script is added |
| Design docs | A table of `Docs/design/*.md` | Generated from their front matter |
| Quick links | Hub board, wiki, skills, MES, WeaponCore, glossary, Workshop pages | The status script's link check |
| Credits | Thank-yous and original authors | Timeless |

## Generated parts
Anything built from data is generated, never typed:
- A generated block sits between `<!-- GENERATED:name -->` and `<!-- /GENERATED:name -->` markers. Its first two lines are an HTML comment and a visible italic line, both saying what generated it, from what, and what to edit instead ("To change it, edit ..."). The visible line is there so a reader knows the table is current and a maintainer doesn't edit it.
- The generator is `tools/Update-Readme.ps1` (or `Update-DocTables.ps1`) in the repo. It rewrites only the marked blocks, and `-Check` reports a block that is out of date or a script, command or file the docs don't cover, without writing, exiting 1 so it can run in a hook or CI.
- Typical generated blocks: a table of core or class limits from the XML, a table of tools or blocks from the SBC, a script table, a command table and the design doc index. The descriptions that data can't hold go in the design docs' front matter (`scripts:`, `commands:`), next to the long explanation.
- Every README, generated or not, is also checked by the GVK Wiki's status script: each backticked repo path (`Content/...`, `ModScripts/...`, `tools/...`) must exist.
- A repo with nothing numeric and fewer than about ten files can skip the generator and keep the standard sections by hand; add one when a table appears.

## Where other content goes
| Content | Home |
| --- | --- |
| How a GVK system is built (the files it uses and why) | `Docs/design/<system>.md` in the repo, next to the files |
| Lookup tables built from data (sandbox variables, prefab tags, encounter types, scripts, tools) | A generated file such as `Docs/reference.md`, rebuilt by a script in `tools/` |
| General lessons that help any server | The `se-dev-mes` skill (or the matching generic skill), kept free of GVK names |
| Current GVK facts agents need while coding | The `se-dev-kharak` skill, kept short |
| Why, history, decisions, season plans | The GVK Wiki |
| Work, ideas and bugs | Hub issues, never Markdown or `TODO` comments |
| Hard-to-reverse decisions | `Docs/adr/` |

## Design docs
A design doc lives in `Docs/design/` (or `docs/design/` where the repo already uses a lowercase `docs/`) and starts with front matter:

```
---
title: Parent leash
notes: [46]
covers:
  - Content/Data/Encounters/GVK-Universal-TriggerGroup-LeashBeacon.sbc
  - tools/New-LeashGroup.ps1
checked-against: MES d85b7b3
checked: 2026-10-26
---
```

`covers` lists the files the doc describes, as paths or globs. `checked` and `checked-against` are the date and the MES build someone last verified the doc against; leave them out if it hasn't been verified. The GVK Wiki's status script flags a doc whose covered files changed after the doc's last commit, so the author sees the drift at the next session instead of in a bug report. `notes` keeps the old README note numbers that hub issues cite; numbers are never reused.

## Never in a README
A limit, price, radius, speed or count that a file holds; the state of an issue or PR; a copy of MES or skill text; a list of what changed (that is the commit log).
