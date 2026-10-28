# Server Context: GV — Deserts of Kharak (GVK)

## 1. Core Philosophy
- **100% Automated Governance**: Every server constraint (zone boundaries, KOTH restrictions, safezone anti-stacking, grid limits) is enforced through active ModAPI systems rather than admin policing.
- **Player-Centric Simplicity**: Straightforward, frictionless rules that let players focus on desert survival and combat without cognitive overload.
- **Lean Architecture (Zero Bloat)**: Keep mod count lean; solve problems with optimized internal code rather than stacking bulky workshop mods.

---

## 2. World & Terrain Design
- **Theme**: Homeworld: Deserts of Kharak single-planet rover-centric PvPvE.
- **Terrain (Pertam Base)**: Custom graded highways, mountain passes, smoothed dunes, and traversable canyons.
- **Voxel Resets**: Full voxel resets daily (fills subterranean holes; bases lower than 50m below surface auto-transferred to SPRT).

---

## 3. Factions & Dynamic Alliance System
- **Core Factions**:
  - `COALITION` (Coalition of the Northern Kiithid, Founder: Rachel S'jet, Starter Hub & Free Scrap Refiners)
  - `GAALSIEN` (Kiith Gaalsien, Founder: Khagaan, Hostile NPC Desert Raiders)
  - `DERELICT` (Hostile automated wreckage & defense relics)
  - `KOTH` (King of the Hill objective faction)
- **Dynamic NPC Alliances**:
  - `SOBAN` (Kiith Soban, Founder: Soban the Red, Neutral mining Kiith)
  - `KHAANEPH` (Khaaneph Scavengers, Clanless southern nomads)
  - Players/Factions choose one-time alignment via `/alliance SOBAN` or `/alliance KHAANEPH`. Territory dynamically expands/contracts based on mission completions and asset destruction.
  - **Territory size** follows each alliance's Influence (`SOBAN_Influence`, `KHAANEPH_Influence`; clamped to 0-1200): under 200 = 10 km, 200-399 = 20 km, 400-599 = 30 km, 600-799 = 40 km, 800-999 = 50 km, 1000 and up = 55 km. Implemented in GVK-Derelicts `GVK-Alliance-Events-ZoneSize.sbc`; every change is announced in chat with its direction.
- **NPC faction relations** (faction-to-faction, set at world build with the commands in §11). Turrets (vanilla and WeaponCore) use these, and so does MES when it rates one NPC against another (MES master after 2.75.02, `67320b7`, PR #461). A pair with none set counts as Enemies, so set all 15:

  | Pair | Value |
  |---|---|
  | GAALSIEN with COALITION, SOBAN, KHAANEPH, DERELICT | -1500 (Enemy) |
  | SOBAN with KHAANEPH | -1500 (Enemy) |
  | COALITION with SOBAN, KHAANEPH | 1500 (Friendly) |
  | DERELICT with COALITION, SOBAN, KHAANEPH | 0 (Neutral) |
  | KOTH with GAALSIEN, COALITION, SOBAN, KHAANEPH, DERELICT | 0 (Neutral) |

  - **KOTH is Neutral to everyone, players included** (static, `StartingReputation` 0). Its weapons that have WeaponCore's Neutrals setting on fire on any grid that breaks that site's rules, NPC or player; which weapons have it on differs per site. Every other NPC prefab keeps Neutrals off so nothing NPC fires on KOTH or on DERELICT.
- **Identity reputation** (GVK-Settings `Factions.sbc`, what MES reads when an NPC rates a player; NPC-vs-NPC uses the table above): COALITION is static at 500 (Friendly to players, to the game and to MES since `67320b7`, PR #460); GAALSIEN and DERELICT are static at -1500; SOBAN and KHAANEPH start everyone at -1500 and stay non-static so AllianceRepHelper can set player reputation; KOTH is static at 0. GVK-Derelicts target profiles skip the factions each one leaves alone by tag, and its override profiles handle forced targets (GVK-Derelicts README note 47).
- **Players and alliances** (AllianceRepHelper, workshop 3660799548, config in the world's `Storage\3660799548.sbm_AllianceRepHelper\AllianceRepHelper.cfg`): a new player faction gets -600 with SOBAN and KHAANEPH; `/alliance <TAG>` sets +1500 with the chosen one and -1500 with the other, for the faction and every member; a joining member copies the faction's values. World setting `ReputationDecayRate` is 0, so nothing decays.

---

## 4. Zone Architecture (Distances from Crossroads Tower: 62495, 28019, 37195)
- **Zone 0 (0 – 20km)**: Safe Starter Hub.
  - Strict PvE; PvP and cross-faction grinding damage blocked via `GVK_NoPVPZone.cs`.
  - Heavy production, large assemblers/refineries, ship drills, and heavy military weapons disabled (`LimitedProdZone_*.cs`). Basic assemblers and basic static drills permitted.
  - Shield Generators are **non-siegable** in Zone 0.
  - 4km border caution: Zone 1 weapons can fire into the edge of Zone 0.
- **Zone 1 (20km – 35km)**: Fledgling PvP & Salvage.
  - Small derelicts/wrecks spawn for active players. Weapons and ship drills enabled; large-scale production disabled.
- **Zone 2 (35km – 50km)**: Contested Desert.
  - Medium defended wrecks, cargo ships, and ancient relics. Large-scale production unlocked.
- **Zone 3 (> 50km)**: Deep Desert / Gaalsien Heart.
  - Heavy military wrecks, Gaalsien convoys/cruisers, relic ammo, and Data Cores. Full uncapped production and warfare.
- **Alliance territory vs Zone 0**: the SOBAN territory centre is 37.2 km and KHAANEPH 47.0 km from Crossroads Tower, so their 40-55 km tiers overlap the starter hub. NPC spawning there stays COALITION-only, but hub players see the alliances' territory enter/leave messages.

---

## 5. Grid Classes, Speeds & Spec Core / Ship Core Framework Limits
A grid's class is determined by its Core Beacon. Points: Utility Points (**UPs**), Mobility Points (**MPs**). Core specifics are reference only, subject to change.
- **Small Grid Core**
- **Large Grid Light**
- **Large Grid Medium**
- **Large Grid Heavy**
- **Large Grid Fortress**
- **Large Grid Pipeline**
- **Large Grid Shield**

### Point Costs & Hard Caps
- **Mobility Points (MPs)**: Suspensions (10 MPs, hard cap 20), Thrusters/Hovers (1 MP, hard cap 200). For reference only, subject to change.
- **Utility Points (UPs)**:
  - Wind/Solar
  - Ship Drills (Reg/Adv)
  - Ship Welders
  - Pistons / Rotors / Hinges
  - Production (Basic/Reg/Adv)
  - Weapons
  - Relic Weapon Slots
  - H2 Engines / H2/O2 Gen
  - Shield Generator
  - SRBM / Odin
  - Drill Blocker

---

## 6. Siegable Shield Generators & Siege Drives (Kamikaze's Mod)
- **Shield Generators**: Replaces vanilla safezones. 250m radius, 1W power, provides free jetpack H2 to owners. Non-siegable in Z0; siegable in Z1–Z3.
- **Siege Drives**: Built on mobile grids within 3km of target Shield Generator. Requires 100 Siege Chips (tokens).
- **Siege Timing**: 30-min drain time (100% to 0%), 15-min recharge time (0% to 100%). Defenders must destroy the attacking Siege Drive (marked with red laser).

---

## 7. KOTH (King of the Hill) Encounters
- **Sites**: Khar Toba (Z3, all grids), Kalash Site (Z3, small grids), Crashed Starship (Z3, small rovers; just inside the Z3 sphere, checked in game 2026-10-26).
- **Anti-Abuse Restrictions**:
  - `KOTHNoThrusters_*.cs`: Shuts off non-NPC thrusters within 3km.
  - `KOTHNoLargeGrid_*.cs`: Shuts off large grid power within 3km.
  - `KOTHNoSafezone_*.cs`: Shuts off player safezones and projectors within range.
  - No digging/drilling or static blocking around KOTH structures.
  - More KOTHs may be added in future seasons.

---

## 8. Logoff, Cleanup & Offline Faction Safezones (FSZ)
- **Offline Faction Safezones (FSZ)**: Automatically protects faction large grids (> 30 blocks) 60 seconds after the last member logs off. Requires no neutral/enemy grids within 1000m.
- **Cleanup Cadence**:
  - Server cleanup every 1 hour (deletes any grid without a beacon).
  - Debris sweep every 30 mins (deletes splits < 3 blocks instantly).
  - Floating objects and ejected connector items deleted immediately.
- **Auto-Hangar**: Inactive faction grids auto-stored 8 days after the last member logged off (an owner setting; may change).

---

## 9. Economy, CUs, RUs & Tech Progression
- **CUs (Construction Units)** & **RUs (Resource Units)**: Core ingots for advanced weaponry and tech.
- **Tech Components**: `[Tech] Igniter`, `[Tech] Grav. Reflector`, `[Tech] Bolt Carrier`, `[Tech] Gun Cradle`, `[Tech] Launch Assem.`, `[Tech] Particle Emit.`, `[Tech] Data Core`, `Turbo Encabulator`.
- **Scrap Refining**: NPC tech grinds into `[Scrap]` Tech; 100% refined into CUs for free at Coalition Base (Z0), Skyport, Mastodon, Sevastopol, or Coalition mobile trade cruisers.

---

## 10. Custom Logistics, Manufacturing & Quality of Life
- **MnM (Manufacturing & Maintenance)**: Custom performant projector welder. 1 active per construct, stationary (< 2 m/s). 1 Uranium = 3x boost for 20m.
- **Pipelines**: Point-to-point wireless logistics hubs (`/pipeline toggle`).
- **Static Drills**: Passive terrain-safe resource wells (50m proximity speed penalty for duplicate ore wells).
- **Grid Defender**: Torch collision plugin suppressing collision damage for grids > 50/100 blocks at < 50 m/s, while preserving high-speed player-made missile kinetic/explosive damage.
- **Dynamic Beacon Signatures**: Beacon range scales dynamically (500m up to max) based on mass, speed, weapon tech, and weather. Grids >= 20k blocks marked as `TopGrid` globally.

---

## 11. Season Operations (Admin)
- **World build:**
  - Set the NPC faction relations from §3 with the CrunchUtils Torch plugin ([TheCrunchy/CrunchUtils](https://github.com/TheCrunchy/CrunchUtils), `Commands.cs`, checked at 23dfcdc). Admin only.
    - `!faction rep change <TagA> <TagB> <amount>` sets (doesn't add) the faction-to-faction reputation: 1500 friendly, 0 neutral, -1500 hostile. The 15 commands for §3:
      ```
      !faction rep change GAALSIEN COALITION -1500
      !faction rep change GAALSIEN SOBAN -1500
      !faction rep change GAALSIEN KHAANEPH -1500
      !faction rep change GAALSIEN DERELICT -1500
      !faction rep change SOBAN KHAANEPH -1500
      !faction rep change COALITION SOBAN 1500
      !faction rep change COALITION KHAANEPH 1500
      !faction rep change DERELICT COALITION 0
      !faction rep change DERELICT SOBAN 0
      !faction rep change DERELICT KHAANEPH 0
      !faction rep change KOTH GAALSIEN 0
      !faction rep change KOTH COALITION 0
      !faction rep change KOTH SOBAN 0
      !faction rep change KOTH KHAANEPH 0
      !faction rep change KOTH DERELICT 0
      ```
    - To check all of them at once, save the world and run se-dev-mes `scripts/save_relations.py <world>\Sandbox.sbc`.
    - `!faction rep <TagA> <TagB>` shows the current value and relation (any player can use it).
    - Never use `all` as `<TagB>`: it sets `<TagA>`'s reputation with every faction and every player identity.
    - `!player rep change` ignores its amount and always sets 1500.
  - Keep `SpawnStaticTown` false while clearing static grids, so they don't all respawn at once; set it true once the trade stations have spawned.
  - Sandbox booleans are set with `/MES.Debug.ChangeBool.<Name>.<true|false>`; GVK-Derelicts' README lists which encounters read each one.
- **About 1-2 weeks into the season:** turn on `LargeWrecksEnabled`.
- **SEDB (Discord bridge):** when SEDB relays server messages, it also picks up MES chat sent with `SendToAllPlayers`. To keep NPC chatter out of Discord, use `[SendToAllOnlinePlayers:false]`, `[IgnoreAntennaRequirement:true]` and a very large override range.
