# Versioning & Release Policy for se-dev-kharak

This project follows **Semantic Versioning 2.0.0** (`MAJOR.MINOR.PATCH`). 

To ensure consistency across human developers and AI coding agents, all future releases and tag updates must adhere to the rules outlined in this document.

---

## 1. Version Number Criteria

### A. PATCH (`1.0.0` ➔ `1.0.1`) — Bug Fixes & Documentation Polish
Bump the **Patch** number for backward-compatible bug fixes, coordinate corrections, and documentation polish.

**Qualifying changes:**
- Correcting typos, broken markdown links, or formatting glitches across `references/`, `SKILL.md`, and `README.md`.
- Minor clarifications or updates to existing rules, coordinate definitions, or guidelines without altering server balance paradigms or adding new reference sections.
- Minor corrections or syntax fixes to ModAPI/Torch C# code snippets in `references/modapi-standards.md`.
- Minor corrections to SBC / ModAdjuster XML snippets in `references/sbc-conventions.md`.

---

### B. MINOR (`1.0.0` ➔ `1.1.0`) — New Features & Expanded Coverage (Backward-Compatible)
Bump the **Minor** number when adding new capabilities, expanding server balance documentation, or introducing new reference guides without breaking existing workflows or repository structure.

**Qualifying changes:**
- **New Reference Guides**: Adding comprehensive guides for additional server systems or integrations in `references/` (e.g., custom logistics, KOTH mechanics, or economy).
- **Server Context Expansion**: Documenting newly added factions, dynamic alliance rules, new grid classes, or zone mechanics (Z0–Z3) in `references/server-context.md`.
- **New ModAPI & Torch Standards**: Introducing new engineering patterns, Torch PatchManager recipes, or Havok/Clang mitigation strategies to `references/modapi-standards.md`.
- **New SBC & ModAdjuster Conventions**: Adding new syntax templates, block definition patterns, or configuration rules to `references/sbc-conventions.md`.
- **New Automation Tools**: Introducing helper scripts or automation utilities for repository management or validation.

---

### C. MAJOR (`1.0.0` ➔ `2.0.0`) — Breaking Changes
Bump the **Major** number when changes break existing workflows, AI agent prompts, or repository structure.

**Qualifying changes:**
- **Repository Restructuring**: Moving, renaming, or deleting core reference files (`references/server-context.md`, `references/modapi-standards.md`, `references/sbc-conventions.md`) or top-level entrypoints that AI agents (Antigravity, Cline, Claude Code) rely on at fixed paths.
- **Major Server Architecture Paradigm Shifts**: Fundamental restructuring of the GVK zone architecture, complete overhaul of grid class UP/MP point systems, or breaking changes to how ModAPI standards or Torch PatchManager patches are designed.
- **Breaking Tool CLI Changes**: Renaming or removing command-line parameters in repository scripts that break existing automated workflows.

---

## 2. Release & Tagging Checklist

Whenever a release is cut, follow these steps sequentially:

1. **Pre-Flight Validation**:
   Verify that all markdown links, code blocks, and references are valid and consistent across `SKILL.md`, `README.md`, and `references/`.

2. **Commit Changes**:
   Stage and commit modified files using conventional commit messages:
   ```bash
   git add -A
   git commit -m "fix: correct coordinate typo in server-context guide"
   # or
   git commit -m "feat: add dedicated KOTH encounter reference guide"
   ```

3. **Tag the Release**:
   Create an annotated Git tag matching the new version:
   ```bash
   git tag -a v1.0.1 -m "v1.0.1 - Documentation polish and coordinate fixes"
   ```

4. **Push Commit and Tag**:
   ```bash
   git push origin main --tags
   ```

5. **Publish GitHub Release**:
   Use the GitHub CLI (`gh`) to publish the release with structured release notes:
   ```powershell
   & "C:\Program Files\GitHub CLI\gh.exe" release create v1.0.1 `
     --title "v1.0.1 — Documentation Polish & Coordinate Fixes" `
     --notes-file "path/to/release_notes.md"
   ```

---

## 3. Golden Rule on Published Tags
> [!IMPORTANT]
> **Never force-push or overwrite a published tag** once public users or automated agents have cloned or downloaded it. Doing so desynchronizes local git clones and breaks package references.
> 
> If a bug or omission is discovered immediately after release, publish a **Patch bump (`v1.0.1`)** rather than moving the existing tag.

