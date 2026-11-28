# Faction paint palette

Paint values for GVK NPC and store grids, and how Space Engineers stores paint. Read this before recoloring a prefab or painting a new NPC grid. History and decisions: GVK Wiki `wiki/systems/npc-grid-paint.md`.

## How SE stores paint [HARD]

A block's paint is `<ColorMaskHSV x="" y="" z="" />` in the prefab SBC. It is not plain HSV but an offset from the color picker's center (`VRage.Game.MyColorPickerConstants`):

| Field | Meaning | From real HSV (0 to 1) |
|---|---|---|
| `x` | Hue | `h` |
| `y` | Saturation offset | `s - 0.8` |
| `z` | Value offset | `v - 0.45` (that is, `- 0.55 + 0.1`) |

- The game clamps `y + 0.8` and `z + 0.45` to 0 to 1, so useful ranges are `y` -0.8 to 0.2 and `z` -0.45 to 0.55.
- The in-game color picker shows `x × 360`, `y × 100`, `z × 100` as its H, S and V.
- To go from a hex color to a mask: convert the hex to HSV, then subtract 0.8 from S and 0.45 from V. Going back, add them and clamp.
- The mask sets the color picker's swatch, not the rendered pixel: the armor texture, the skin (`Battered_Armor`, `SciFi_Armor`, `Dust_Armor`...) and the lighting all change how it looks. Compare in game, side by side, before calling two grids the same color. For example, full black (`0, -0.8, -0.45`) with `Battered_Armor` looks medium grey in game (owner, 2026-11-28).
- **Hue alone doesn't make a color.** A mask with a yellow hue but `y` near -0.7 renders as a warm grey (`0.097, -0.72, -0.13` is #524F4B), and a dark `z` renders brown (`0.122, -0.05, -0.15` is #4D3D13). Classify shades by all three values.

## Palette [SOFT]

The owner's reference palette for the Northern Kiithid (owner, 2026-11-28). It was compiled from Deserts of Kharak renders and community color lists, not read from game files, so treat the hex values as the chosen target, not as Blackbird's data. The Coalition orange was sampled from the owner's swatches of the game's fleet render (average of #DC591D lit and #D05419 shaded).

| Kiith / use | Role | Name | Hex | ColorMaskHSV `x, y, z` | Picker H, S, V |
|---|---|---|---|---|---|
| S'jet (COALITION) | Primary hull | Ochre Yellow | #D49B2A | 0.111, 0.002, 0.381 | 40, 0, 38 |
| S'jet (COALITION) | Trim | Dark Charcoal | #2B2D2F | 0.583, -0.715, -0.266 | 210, -72, -27 |
| S'jet (COALITION) | Markings | Off-White | #E6E4DD | 0.13, -0.761, 0.452 | 47, -76, 45 |
| S'jet (COALITION) | Accent | Coalition Orange | #D6571B | 0.053, 0.074, 0.389 | 19, 7, 39 |
| S'jet (COALITION) | Undercarriage, suspension, wheels | Black, `Dust_Armor` skin | #000000 | 0, -0.8, -0.45 | 0, -80, -45 |
| Soban | Primary hull | Crimson Red | #9E2A2B | 0.999, -0.066, 0.17 | 360, -7, 17 |
| Soban | Trim | Gunmetal Grey | #333538 | 0.6, -0.711, -0.23 | 216, -71, -23 |
| Soban | Markings | Warm Sand | #D9D2C5 | 0.108, -0.708, 0.401 | 39, -71, 40 |
| Siidim | Primary hull | Slate Navy | #222D3D | 0.599, -0.357, -0.211 | 216, -36, -21 |
| Siidim | Trim | Carbon Black | #1C1E21 | 0.6, -0.648, -0.321 | 216, -65, -32 |
| Siidim | Markings | Crisp White | #EBEBEB | 0.0, -0.8, 0.472 | 0, -80, 47 |
| Naabal (industrial) | Primary hull | Mustard Tan | #C2A356 | 0.119, -0.243, 0.311 | 43, -24, 31 |
| Naabal (industrial) | Trim | Industrial Grey | #595D60 | 0.571, -0.727, -0.074 | 206, -73, -7 |
| Naabal (industrial) | Markings | Pale Yellow | #E3C878 | 0.125, -0.329, 0.44 | 45, -33, 44 |
| Rachel S'jet (science, her Baserunner) | Primary hull | Arctic White | #E8E6DF | 0.13, -0.761, 0.46 | 47, -76, 46 |
| Rachel S'jet (science, her Baserunner) | Trim | Signal Red | #B8332A | 0.011, -0.028, 0.272 | 4, -3, 27 |
| Rachel S'jet (science, her Baserunner) | Markings | Dark Slate | #2F3236 | 0.595, -0.67, -0.238 | 214, -67, -24 |
| Northern Command (UI, emblems) | Primary | Tactical Cyan | #3BAFDA | 0.545, -0.071, 0.405 | 196, -7, 40 |
| Northern Command (UI, emblems) | Secondary | Deep Navy | #0F1926 | 0.594, -0.195, -0.301 | 214, -20, -30 |
| Northern Command (UI, emblems) | Accent | Ice Blue | #A7D6F1 | 0.561, -0.493, 0.495 | 202, -49, 50 |

- In the game's desert light, S'jet yellow reads warmer (about #DFA837) and Soban red rustier (about #8C322B). Coalition engine trails and highlights are bright cyan-blue (#2299DD to #4EADE4).
- Applied so far: COALITION grids are being moved to the S'jet row (hub #823). The other rows are reference only; GAALSIEN, KHAANEPH and DERELICT have no row.
- Black is for running gear under `Dust_Armor`. Where a COALITION surface should read dark grey under `Battered_Armor`, paint it Dark Charcoal, not full black (owner, 2026-11-28; hub #823).
- COALITION wheels are sometimes primary yellow instead of black, when the source material shows them that way. Keep a wheel's existing role when correcting colors (owner, 2026-11-28).

## Other GVK colors in use (owner, 2026-11-28)

| Use | Hex | ColorMaskHSV `x, y, z` |
|---|---|---|
| KHAANEPH main hull brown | #401600 | 0.058, 0.2, -0.2 |
| Cargo container stacks (COALITION stations) | #5E2F22 | 0.036, -0.16, -0.08 |

KHAANEPH-style grids also carry `0.158, -0.48, 0.12` (#918F63, khaki) and `0.05, 0.1, 0.11` (#8F350E), the colors `GVK-Alliance-ToNeutralColor-Manipulation` repaints. The Caiman, Croc and Gator rovers and the Siege Cruisers in the store use this Khaaneph set, not the S'jet one [SOFT: read from the prefabs].

## Paint style (owner, 2026-11-28)

The owner paints most NPCs this way, each in its faction's colors:
- `Battered_Armor` skin on most armor surfaces.
- `SciFi_Armor` touches, mostly on the dark grey areas, for contrast.
- Non-armor blocks get clean paint (no skin) in the same color, because they look bad in SciFi and blend in better clean.

A color correction changes `ColorMaskHSV` only; it leaves the skins (`SkinSubtypeId`) alone unless the owner asks for a paint change too.

## Recoloring prefabs

- Edit `ColorMaskHSV` in the prefab SBC text. Blueprints stored inside projectors carry their own copies, so change those too. Keep CRLF and the BOM.
- MES `[RecolorGrid:true]` with `[RecolorOld:]`/`[RecolorNew:]` pairs in a manipulation profile repaints a grid as it spawns. It only reaches grids MES spawns, not store grids or projector copies, so use it for per-spawn variants (an alliance's colors), not to fix a prefab's base colors.
- Store grids have preview icons (GVK_Derelicts `Content/Textures/GUI/Prefabs`). Re-render the icon after recoloring a store grid.
