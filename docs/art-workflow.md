# Art workflow (without hand-drawn pixel art)

The user does not draw pixel art. The workflow must produce sprites from
generators or 3D renders, with automatic cleanup, and keep manual work to
choosing results and erasing stray pixels.

Wargroove constraints (official modding guide): animated sprites are
Aseprite files with named tags and a pivot in a `.meta` file; units and
portraits must use only the Wargroove palette, and the mod packer rejects
any off-palette pixel.

## Option A — 3D render to pixel art (best consistency)

1. Get a 3D model: free or paid asset, or generated (AI image-to-3D), kept
   low-poly.
2. Rig and animate it with Mixamo (free auto-rig and animation library;
   use "In Place" animations).
3. Render in Blender with an orthographic camera at the game's angle, then
   pixelate (add-ons such as "True Pixel Art Generator", or Retro
   Diffusion's "Blender to Pixels" tool), with an outline and fixed pixel
   scale.
4. Automatic cleanup script: snap colours to the Wargroove palette, binary
   transparency, crop and align frames on the pivot.
5. Assemble the frames into an Aseprite file with tags through Aseprite's
   command line (batch mode and Lua scripting; exact calls to verify).

Strengths: every frame is consistent because the same model moves;
vehicles and drones are easy (rigid objects). Cost: learning Blender basics
once; human figures need rigged models.

## Option B — AI generation (fastest to start)

1. Generate a base sprite (Retro Diffusion, PixelLab) at the target size.
2. Animate with PixelLab's animation tools.
3. Same cleanup script and Aseprite assembly as option A.

Strengths: quick results. Weakness: frames of an animation drift from one
another; works best for static images (portraits, icons, props) and short
animations.

## Option C — Commission a pixel artist

For the few sprites that matter most (the player's commander, the faction's
signature unit). Cost to estimate with artists directly.

## Recommendation

- Vehicles and drones: option A.
- Portraits, icons, props: option B.
- Soldiers: try A (Mixamo) and B on one unit, keep the better one.
- Prove the chosen workflow on **one unit** through the mod packer before
  producing the roster (roadmap milestone M3).

## Tools Claude can provide

- Colour-snapping and frame-alignment script (Python).
- Aseprite command-line script to build tagged files from frames.
- A checker that reports off-palette pixels before running the packer.

## Sources

- Wargroove modding guide (art section), summarised in `engines/wargroove.md`
- https://www.aseprite.org/cli/
- https://astropulse.itch.io/blender-to-pixels
- https://superhivemarket.com/products/true-pixel-art-generator-v2/docs
- https://gamedev.tv/courses/blender-sprites
- https://forum.freegamedev.net/viewtopic.php?p=72603
