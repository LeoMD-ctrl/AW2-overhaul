# Sprite prototypes (PixelLab) — specification

Prototypes come **before** the final art. Their job is to validate the look
of the modern faction: silhouette, readability, role identification and fit
with Wargroove's style. They are not final assets: sizes, frame counts and
palette will be adjusted once the game's own sprites are measured (after the
game is bought).

## What is known and what is not

- Known (official modding guide): animated sprites are Aseprite files with
  named tags and a pivot; units and portraits must use only the Wargroove
  palette; the packer rejects off-palette pixels.
- Known (palette file seen in a published mod): the palette image is
  256 × 16; its first row holds about 240 reference colours, other rows hold
  variants (probably army and skin colours — to confirm).
- **Not known yet:** the pixel size of map sprites and battle sprites, frame
  counts per animation, facing direction conventions. Measure them in
  `modpacker_bin/original_assets` before producing final art.

## Order of work

1. MANPADS (first spike unit)
2. Drone
3. Rifleman (basic infantry of the faction)
4. Engineer
5. Light camp (structure)
6. Commander portrait

One unit at a time: do not start the next before the previous passed the
review checklist below.

## Deliverables per unit (prototype stage)

| # | Image | Purpose | Prototype canvas |
|---|---|---|---|
| 1 | Concept, three-quarter view | Agree on the look | free size, high detail |
| 2 | Map sprite, idle, 2 frames | Readability on the map | 64 × 64 canvas, character about 24–32 px tall |
| 3 | Battle sprite, idle, 1 frame | Look in the battle scene | 128 × 128 canvas |
| 4 | Battle sprite, attack key frame, 1 frame | Readability of the action | 128 × 128 canvas |
| 5 | Portrait (characters only) | Dialogue scenes | 128 × 128 canvas |

Canvas sizes are working sizes for prototypes, not the game's sizes.

## Visual rules

- Pixel art with **no anti-aliasing, no blur, no gradients**; every pixel
  fully opaque or fully transparent.
- **Dark outline** around the silhouette, flat shading in 2–3 tones per
  material, one light direction (top-left) for every unit.
- **Army colour zone:** paint the parts that should take the player's colour
  (uniform, vehicle panels) in **one dedicated hue ramp** (for example pure
  reds, 3–4 tones) used nowhere else. It will be mapped to Wargroove's army
  colours later.
- **Skin** in its own ramp, used only for skin.
- Keep the total number of colours per sprite small (aim for 16 or fewer).
- **Role readable at a glance:** the MANPADS tube, the drone's rotors, the
  engineer's tools must be visible at map size.
- Modern military gear, but proportions and colour energy close to
  Wargroove's units (compare side by side with screenshots of the game).
- Map sprites face **right** by default (to confirm with the game's files).

## Files and naming

- Folder: `art/prototypes/wg2/<unit>/` in this repository (only our own
  generated images; never the game's files).
- Name: `<unit>_<view>_<state>_v<NN>.png`, e.g.
  `manpads_map_idle_v03.png`, `manpads_battle_attack_v01.png`.
- Next to the images, `prompts.md`: the prompt, the tool, the settings
  (size, palette, seed, reference images) and a note on what worked. This
  also feeds the prompt library of the Obsidian vault.
- Check PixelLab's terms on ownership of generated images before
  publishing them.

## Review checklist (before moving to the next unit)

- [ ] Silhouette readable at 100 % zoom; role identifiable without colour
- [ ] Army colour zone in a single dedicated ramp, nothing else in that hue
- [ ] Dark outline closed around the silhouette
- [ ] No semi-transparent pixels, no isolated stray pixels
- [ ] 16 colours or fewer
- [ ] Light from the top-left, consistent with the other units
- [ ] Looks coherent next to a Wargroove unit screenshot at the same scale
- [ ] Prompt and settings saved in `prompts.md`

## Example prompts (PixelLab)

MANPADS, map sprite:

```
pixel art game unit, modern soldier kneeling with a shoulder-fired
anti-aircraft missile launcher aimed upward, tactical helmet, plate
carrier, side view three-quarter facing right, chunky readable silhouette,
bold dark outline, flat shading, limited palette, uniform in red tones only,
transparent background, no anti-aliasing, fantasy tactics game style
```

Drone, map sprite:

```
pixel art game unit, small military quadcopter drone with a camera pod and
a light machine gun, hovering, three-quarter view facing right, chunky
readable silhouette, bold dark outline, flat shading, limited palette,
body panels in red tones only, transparent background, no anti-aliasing,
fantasy tactics game style
```

## From prototype to final

1. Measure the game's sprite sizes, frame counts and pivots.
2. Adjust the approved prototype to those sizes.
3. Produce the animations (art workflow: generation or 3D render).
4. Snap colours to the Wargroove palette (army ramp → army colours),
   assemble the Aseprite file with tags, run the packer.
