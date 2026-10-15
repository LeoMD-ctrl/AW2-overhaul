# Sprite specification

Everything below was measured on the Commander Wars assets or read in its
source. Rendering quality (especially higher-resolution sprites) still has to
be checked in game.

Reference folders in Commander_Wars:
`resources/images/units/`, `resources/images/battleanimations/`,
`resources/images/building/`, `resources/images/terrain/`, each with a
`res.xml` declaring every image.

## 1. Golden rules (all images)

- **PNG with transparency.** Background fully transparent, and every
  pixel either fully opaque or fully transparent: the base unit sprites use
  alpha 0 or 255 only.
- **No anti-aliasing, no blur, no smooth resize.** Every resize uses
  nearest-neighbour. The atlases use `linearFilter="false"`.
- **Exact colours only in coded images** (see 2). One wrong RGB value =
  one wrong pixel in game, because colours are looked up, not used as is.
- **Frames on an exact grid:** a sheet is `cols × rows` frames of identical
  size, with no margin and no spacing.
- **File names follow the base game:** `<id>+mask.png`, `<id>+walk+mask.png`,
  etc., and every image is declared in the mod's `res.xml`.

## 2. How colours work: coded images

Units, buildings and terrain are not drawn in their final colours. They are
drawn in **code colours**; the engine replaces each code by a real colour
from a table (`system/frac_matrix_shader.glsl`): the pixel's **red and green
values are the coordinates** of the final colour in a colour table.

- **Units and buildings (`+mask` images):** the table is the player's army
  table (`resources/images/colortables/orange_star.png`, `blue_moon.png`…,
  256 × 256). The same sprite therefore shows in every army's colours.
  The base game's infantry mask uses 14 code colours for the whole unit:
  team colours and neutral colours (skin, grey…) are all codes.
- **The palette to use is `resources/images/colortables/Color_Codes.png`:**
  the base game's reference sheet, with code ramps per category
  (primary / secondary / tertiary colours; base, infantry, recon, ship;
  unit, land vehicle, air vehicle, large). Draw only with these colours.
- **Images without `+mask`** (for example `infantry+os.png`) are drawn as
  is (`Recoloring_None`) on top of the mask: free colours for details that
  must not change with the army.
- **Terrain:** same principle with a 64 × 64 biome palette
  (`resources/images/terrain/biome_palettes/`, e.g. `palette_clear+aw2.png`
  plus `+dawn`, `+dusk`, `+night` variants). The terrain codes are not the
  same as the unit codes; the exact list must be extracted before drawing
  terrain.

**Practical workflow:** draw in normal colours with the locked palette
`art/palettes/aw2o_units.gpl` (the 22 codes the base unit masks actually
use, shown in Orange Star colours), then convert each colour to its code
(`art/palettes/aw2o_units_codes.json`). See `art/README.md`.

## 3. Units

### 3.1 Map sprite (required)

| File | Layout | Example |
|---|---|---|
| `<id>+mask.png` | 4 frames side by side (idle animation), `cols = "4"` | `infantry+os+mask.png`: 128 × 32 → frames of 32 × 32 |
| `<id>.png` (optional) | same layout, free colours, drawn on top | |

The base game draws units at **32 × 32 per frame**. Larger frames (64 × 64)
are accepted: the engine scales each frame to the tile size
(`game/unit.cpp`).

**Higher resolution, per sprite type (verified in the source):**
- map sprite: any frame size, scaled automatically to one tile;
- walk sprite: the unit script passes a scale to
  `animation.loadSpriteV2(id, mode, scale)` (the base game already uses 1 or
  2 per army style, `animationData` in `infantry.js`), so 96 × 96 frames
  with a scale of 0.5 take the place of 48 × 48 ones;
- battle animation: `scale` parameter of the battle sprite functions;
- terrain and buildings: scaled by frame width.
On screen the sprite still covers one tile; at the default zoom a 2× sprite
is reduced without smoothing, so 1-pixel details may vanish. Test one HD
sprite in game before producing a full set.

### 3.2 Movement sprite (required)

`<id>+walk+mask.png`: **4 rows = 4 directions**, N frames per row.
Row order (`game/gameanimation/gameanimationwalk.cpp`):

| Row | Direction |
|---|---|
| 0 | South (towards the viewer) |
| 1 | North |
| 2 | East |
| 3 | West |

Measured sizes:

| Unit | Sheet | Frames |
|---|---|---|
| Infantry | 192 × 192, `cols="4" rows="4"` | 48 × 48 |
| Recon | 144 × 192, `cols="3" rows="4"` | 48 × 48 |
| Transport copter | 128 × 256, `cols="2" rows="4"` | 64 × 64 |

### 3.3 Battle animation (optional at first)

Played in the detailed battle view. One horizontal strip per state, sizes
depend on the unit:

| Unit | Example | Frame |
|---|---|---|
| Infantry | `infantry+ac+fire+mask.png` 64 × 32 (2 frames) | 32 × 32 |
| Recon | `recon+ac+mask.png` 49 × 56 | 49 × 56 |
| Transport copter | `t_heli+os+mask.png` 256 × 104 (2 frames) | 128 × 104 |

Infantry has states such as idle, walk, stop, fire, fire up / down, dying,
and river variants (228 files for all armies). A script
(`resources/scripts/battleanimations/BATTLEANIMATION_<ID>.js`) chooses
sprites, scale, frame count, frame time and effects
(`game/gameanimation/battleanimationsprite.h`), so larger or longer
animations are possible.

**Until the art exists, a new unit reuses an existing battle animation**
(placeholder), so it can be tested without it. The battle view also has an
"overworld" mode that only uses map sprites.

### 3.4 Army styles

Base units have one sprite set per army style (`os`, `bm`, `ge`, `yc`, `bh`,
`bg`, `ma`, `ac`, `bd`, `dm`, `gs`, `pf`, `ti`), mapped in the unit script
(`armyData` in `infantry.js`). **Our units start with a single style for
every army**: the colours still change per army through the mask.

## 4. Buildings

Measured on the town and the airport:

| File | Size | Notes |
|---|---|---|
| `<id>.png` | 32 × 32, `cols="2"` | 2 frames of **16 × 32** (1 tile wide, 2 tiles high) |
| `<id>+mask.png` | 16 × 32 | coded colours, recoloured by owner |
| `<id>+neutral.png` | 16 × 32 | neutral (unowned) version |
| `<id>+snow.png` | 16 × 32 | snow variant |

The light camp can start from the temporary airport set
(`temporary_airport`, `+mask`, `+neutral`, `+snow`). Buildings, like terrain,
are drawn at **16 px per tile** in the base game; 32 × 64 frames are accepted
(the engine scales by frame width, `game/building.cpp`).

## 5. Terrain and a more detailed map

- Base terrain tiles are **16 × 16** (`plains+0.png`…), animated or edge
  variants as horizontal strips (`sea+E+S+W+mask.png`: 128 × 16, 8 frames).
- Each terrain has many edge variants named after its neighbours (`sea` has
  115 files), plus biome folders (`desert`, `snow`…) and the biome palettes
  (`aw1`, `aw2`, `awds`, × dawn / dusk / night).
- **More detail, three levers:**
  1. **Redraw tiles at 32 × 32** (twice the base detail): the engine scales
     each frame to the tile size (`game/terrain.cpp`). Big job: every edge
     variant of a terrain must be redrawn, and the biome palettes must keep
     working. Start with one terrain (plains, then forest) to validate.
  2. **Decorations per tile in the map editor:** the editor can place
     overlay sprites and custom images on a tile (`loadOverlaySprite`,
     `customTerrainImages/` folder of the user, "modify terrain" dialog).
     No code needed; good for unique details (ruins, wrecks, craters).
  3. **Redraw buildings at 32 × 64.**
- Units are already at 32 px: on a redrawn map, units and terrain end up at
  the same level of detail.

## 6. In the mod

```
images/
  units/res.xml            declares our unit sheets
  units/manpads/...        one folder per unit
  battleanimations/res.xml
  building/res.xml
  terrain/res.xml
```

Each `res.xml` follows the base format
(`<resources><set path="..."/><atlas linearFilter="false"><image file="..." cols="4"/>…`).
A mod image with the same id as a base image replaces it
(see `mods/16x16_units`, `mods/aw_styled_icons` in Commander_Wars).

## 7. Tools

- **Generation:** PixelLab, Retro Diffusion. They give a base; expect manual
  cleanup.
- **Cleanup and animation:** Aseprite (paid) or LibreSprite (free fork):
  grid, palette lock, indexed colour mode, sheet export.
- **Planned script in this repository:** convert a sprite drawn in Orange
  Star colours to code colours, and check frame sizes, transparency and
  colours against this specification.

## Deliverables for the planned units

| Unit | Map (`+mask`) | Walk (`+walk+mask`) | Battle | Reference |
|---|---|---|---|---|
| MANPADS | 4 × 32 × 32 | 4 × 4 × 48 × 48 | later (infantry placeholder) | infantry |
| Engineer | 4 × 32 × 32 | 4 × 4 × 48 × 48 | later (infantry placeholder) | infantry |
| Drone | 4 × 32 × 32 | N × 4 × 64 × 64 | later (copter placeholder) | transport copter |
| Light camp | building set, 16 × 32 (or 32 × 64) | — | — | temporary airport |
