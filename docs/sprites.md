# Sprite formats (measured on the Commander Wars assets)

Reference folders in Commander_Wars:
`resources/images/units/`, `resources/images/battleanimations/`,
`resources/images/building/`, each with a `res.xml` that declares every image.

## Map sprite of a unit

Example: `resources/images/units/infantry/`.

| File | Size | Layout |
|---|---|---|
| `infantry+os+mask.png` | 128 × 32 | 4 frames of 32 × 32 (idle animation), `cols = "4"` |
| `infantry+os+walk+mask.png` | 192 × 192 | 4 × 4 frames of 48 × 48 (walking), `cols = "4" rows = "4"` |

- `os`, `bm`, `ge`, `yc`, `bh`… are army styles; a unit script maps each
  army to a sprite set (`armyData` in `resources/scripts/units/infantry.js`).
  A new unit can use a single style for every army to start with.
- `+mask` images are recoloured with the player's colour
  (`GameEnums.Recoloring_Matrix`); images without `+mask` are drawn as is
  (`Recoloring_None`). The colour tables are in
  `resources/images/colortables/`. **To do before producing many assets:**
  confirm the exact palette a mask must use, by starting from an existing
  mask as a template.
- No smoothing: atlases use `linearFilter="false"`, so pixel art stays
  sharp.

## Battle animation

Example: `resources/images/battleanimations/infantry/` (228 files).
Frames are 32 × 32; animations are horizontal strips, e.g.
`infantry+ac+fire+mask.png` is 64 × 32 (2 frames). There are separate
strips per state (idle, walk, stop, fire, fire up / down, dying, river
variants). The script that plays them is
`resources/scripts/battleanimations/BATTLEANIMATION_INFANTRY.js`.

## Buildings

`resources/images/building/` (e.g. `airport.png`, `airport+mask.png`,
`airport+neutral.png`, `airport+snow.png`). Sizes to measure when we work
on the light camp.

## In the mod

Images go under `images/<category>/` in this repository, with their own
`res.xml` declaring them (see `mods/aw_styled_icons/images/` in
Commander_Wars for a mod that ships images).
