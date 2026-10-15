# Art kit

Tools to draw sprites that follow `docs/sprites.md`.

## Palettes (`palettes/`)

| File | Use |
|---|---|
| `aw2o_units.gpl` | Drawing palette for Aseprite (load it from the palette menu). 22 colours, in Orange Star colours. |
| `aw2o_units_palette.png` | Same palette as a 22 × 1 PNG (Aseprite can also load it). |
| `aw2o_units_swatch.png` | Readable sheet: role of each colour and whether it changes with the army. |
| `aw2o_units_codes.json` | Orange Star colour → engine code colour. Used to convert drawings to `+mask` images. |

The 22 colours are the codes actually used by the base game's unit masks
(measured on `resources/images/units/**/*+mask.png`), shown through the
Orange Star colour table: a 7-step army colour ramp plus white, 3 greys that
change with the army, and fixed colours (greys, tan, skin, glass, green).
Draw only with these colours.

## Templates (`templates/`)

Each `*.png` is a reference layer at the exact size (magenta = frame
border, cyan dashes = one map tile). Each `*_preview.png` is an enlarged,
labelled view.

| Template | Content |
|---|---|
| `unit_map_32` | Map sprite: 4 idle frames, 32 × 32 |
| `unit_walk_48` | Walk: rows South, North, East, West × 4 frames, 48 × 48 |
| `air_walk_64` | Air walk: 4 rows × 2 frames, 64 × 64 |
| `building_16x32` | Building: 16 × 32 |
| `hd_unit_map_64`, `hd_unit_walk_96` | Double-resolution versions, **to test in game first** |

## Rules that matter most

- Pixels are fully opaque or fully transparent: the base sprites use alpha
  0 or 255 only.
- A 1-pixel outline in the darkest army colour surrounds the silhouette.
- Map sprites face right; feet on the bottom edge of the frame.
- No anti-aliasing, no blur, nearest-neighbour resize only.

## Workflow

1. Generate a base (Retro Diffusion), refine and animate (PixelLab).
2. In Aseprite: load `aw2o_units.gpl`, set the grid to the frame size, paste
   the frames, convert to indexed colours with this palette to snap every
   pixel to it, clean up, export a sheet with no padding.
3. Commit the PNG (in Orange Star colours) under `art/source/`; it is then
   converted to code colours for the game.

`make_art.py` (kept outside the repository) generated these files from the
Commander Wars assets.
