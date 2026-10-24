# Owner Workflow: MES Contributions, In-Game Tests, Project Rules

How the GVK server owner wants upstream MES work, in-game tests and GVK project decisions handled. These rules apply on top of the `se-dev-mes-contributing` and `se-dev-mes` skills, and win where they differ. `se-dev-mes-contributing` belongs to VillanorSE: never edit it; record changes here instead.

---

## 1. MES Upstream Contributions

### When an MES change counts as available
- **Merged into MeridiusIX master is enough.** GVK content may rely on it, and the `se-dev-mes` tag cache and docs may include it, labelled "master after <version>". It doesn't have to be on the Workshop yet.
- **Never build on an unmerged PR or fork branch**, in GVK content or in `se-dev-mes`. Rebuild the tag cache from `upstream/master`, not from the local clone's current PR branch.
- When GVK content needs a master-only change, say so in the commit or README note: a server on the Workshop MES won't have it until the next release.

### Posting
- Draft first. Nothing goes upstream (issues, comments, PRs, review replies) until the owner says "post it". A comment about the evidence is not approval.
- Settle ambiguous causes with a code check, not an in-game test, then report and wait.

### Issue, PR and comment text
- **No code blocks**, not even the proposed change: describe it in a sentence and link where it goes. Two or three permalinks at most.
- **Short**: about 100 words for an issue, one sentence per point (problem, link, sibling link, seen in game, fix plus severity, PR offer).
- **Titles** are short and named for what a modder sees, not code internals ("RecolorGrid action can fail after part of the grid breaks off").
- **Bullets** are one short line each. If a point needs several sentences, give it a plain sentence after the list. Never put a permalink inside a bullet.
- **Write for a maintainer who wasn't part of the test**: name the target and setup up front ("with my character as the target"), not "the grid's side of me".
- **Plain language** over precise jargon: one sentence per difference the reader needs ("rounds both colors to 3 decimals, so a typed 0.333 matches a block saved as 0.333333343").
- Inline backticks for tag names are fine in moderation: mark the tags the reader needs to spot, leave repeats plain.
- In a comment on someone else's issue, leave links out when the explanation already makes the point.
- Separate the smallest fix from your preferred one when proposing more than the minimum ("The smallest fix would be X. I'd propose this instead: ...").

### PR code
- **No new comments** unless they continue a pattern the file already has (e.g. the `//TagName` line above each tag parse); enenra dislikes comments in code. Put the reasoning in the PR description.
- **Never delete existing comments**, including commented-out code, unless the code they describe is gone.
- **Keep the diff minimal**: gate existing code in place with a flag instead of moving blocks. (MES #449 went from 17+/38- to 8+/1- this way.)
- Check before committing: `git diff master..HEAD | grep "^[+-].*//"`.

### During review
- Answer review comments with **new commits and a normal push**. Don't rebase or force-push while GitHub shows the PR as mergeable: it marks the maintainer's threads outdated and breaks "changes since your last review". Squashing before the first review is fine.
- Reply to each thread, resolve it, then re-request review. Re-requesting only works from the GitHub website (the circular arrow); the API refuses for this account.

---

## 2. In-Game Tests (MES)

- **Confirm the deployed copy first.** GVK repo edits only reach the game after an MDK2 build to `%APPDATA%\SpaceEngineers\Mods`; check the deployed files have the change before handing over a test. The local MES clone is live without a build.
- **Follow `se-dev-mes` `references/testing_and_debugging.md`**: `/MES.IGGB` and `/MES.Debug.DrawPaths` first, then `[DebugMessage:]`, then logs. Give a pass/fail the tester can read (a GPS or IGGB value that must match, a chat line), not "it seems to follow".
- **Reuse production autopilots and behaviors** for test grids; only build a test profile when the test is about that profile.
- **Nearby test spawns**: a test-only spawn group with `[DroneEncounter:true]`, `[AdminSpawnOnly:true]` and Min/MaxDroneDistance spawns a few hundred metres away; `PlanetaryInstallation` lands about 5 km away. Put test-only files (`ZZ-Test-*.sbc`) and test-only tag changes in the deployed copy only; the next build removes them.
- **Minimise reloads**: put every case that needs the same config into one load, and include a control (a profile left at the old value) where it costs one extra spawn.
- **Test world "GV Kharak S11 NPC Dev"**: Space Cargo Ships, Random Encounters and Boss Encounters have EnableSpawns false (Drone Encounters and Planetary Cargo Ships are on). Enable a type with `/MES.Settings.<Type>.EnableSpawns.true` and turn it off after. A test group's Frequency is multiplied by 10 and rounded, so use `1.0` (under `0.05` never spawns, even by name). Verify every chat command against `ChatMessage.cs`. The world has many other NPCs; `/MES.Debug.RemoveAllNpcs` clears them (destructive, offer it as optional). Admin Invulnerable (not Untargetable) keeps the tester alive as a target.
- **WeaponCore ignores vanilla `<TargetingGroups>`** (it picks subsystem targets by block class). Every local MES world also loads WeaponCore, so vanilla targeting needs a fresh vanilla + MES world.

---

## 3. Project Rules

- **Zones**: the `[MES Zone]` profiles in GVK-Derelicts (`GVK-Universal-Zone.sbc`) are the source of truth for Z0-Z3. The zone nav HUD and any other display follow them; when they disagree, change the display.
- **Server not in production** (as of 2026-10-11): rename spawn groups, behaviors, prefabs and IDs outright, with no legacy aliases or "next wipe" deferrals. Mention a real save-compatibility consequence in one line if it matters for launch. Re-check only if the owner says the server is live.
- **Hub issues**: record a settled decision on its issue in the same session (comment or tick the box). Add every player-facing change to the season's Steam guide issue (Season 11: GV-Server-Mods/GVK-Settings#645) and say so in the reply.
