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

## Higher-resolution sprites (verified in the source, not tested)

- **Map sprites (units, terrain):** the engine scales each frame to the tile
  size: scale = tile size (32) ÷ frame width (`game/unit.cpp`,
  `game/terrain.cpp`). A 64 × 64 frame is therefore drawn in the same
  square as a 32 × 32 one; the bundled `mods/16x16_units` mod uses this the
  other way round, with 16 × 16 frames. More detail only shows when the map
  is zoomed in; how it looks must be checked in game.
- **Battle animations:** the animation script chooses the sprite, its
  `scale`, the number of frames, the frame time and the position
  (`loadSpriteV2` / `loadMovingSpriteV2` in
  `game/gameanimation/battleanimationsprite.h`). A mod can thus ship
  larger frames loaded at a smaller scale, more frames, or new effects
  (shake, colour flash: `addUnitshakeToLastLoadedFrame`,
  `loadColorOverlayForLastLoadedFrame`).
- **Frame count:** set by `cols` / `rows` in `res.xml` for map sprites, and
  by the strip length (or the `frames` parameter) for battle animations.
- Replacing an existing sprite: a mod ships an image with the same id in its
  own `res.xml`, like `mods/16x16_units` and `mods/aw_styled_icons`.

## In the mod

Images go under `images/<category>/` in this repository, with their own
`res.xml` declaring them (see `mods/aw_styled_icons/images/` in
Commander_Wars for a mod that ships images).
