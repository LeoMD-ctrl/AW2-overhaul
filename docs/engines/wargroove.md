# Engine notes: Wargroove

Candidate engine, reviewed 2026-10-15. The user owns neither Wargroove nor
Wargroove 2. Facts below come from the official modding guide (pasted by
the user), the source of published mods and official blog posts.

## Verified in published mods

Read from `Innocience/UnitRanks` and `wargroove-modding/Pet-The-Dog`
(GitHub):

- **Mod layout:** `manifest.yaml` (id, name, author, description),
  `assets_src/config/...` (YAML), `assets_src/lua/...`, `assets_src/image/...`,
  `assets_src/audio/...`, packed into `assets/mod.dat` and
  `assets/modAssets.dat`.
- **Modify existing units in YAML:** `modifyingUnitClasses` with `add`
  (e.g. add a verb to `soldier`, `archer`…) or `replace` (e.g. replace a
  unit's `grooveId`).
- **New unit actions ("verbs") in Lua:** a verb is declared in `verbs.yaml`
  (`id`, `icon`, `targetted`, `scriptName`, `aiUses: true`) and implemented
  in Lua (`Verb:new()`, `canExecuteWithTarget`, `execute`). The same file
  gives the AI its behaviour: `generateOrders` lists possible orders and
  `getScore` rates them, so **the AI can use a new action**.
- **Per-unit data:** `Wargroove.setUnitState(unit, key, value)` /
  `getUnitState` (UnitRanks stores experience and rank this way).
- **Override combat logic:** UnitRanks replaces `Combat:solveCombat` and
  `Combat:getBestWeapon`, and all of `events.lua`.
- **New trigger actions for the map editor:** declared in
  `config/gameplay/triggers/actions.yaml` (e.g. `modify_experience`,
  `modify_rank`) with typed parameters.
- **Art:** mods ship Aseprite source files (`.ase`) with a `.meta` file for
  the pivot point; UnitRanks includes a 256 × 16 `wargroove_palette.png`.
- **Precedent for axis 1:** UnitRanks already gives units XP equal to the
  economic damage dealt, 4 ranks, +10 % damage per rank, with the next-rank
  progress shown in the groove UI, without new unit art (only rank icons).

Lua functions seen in these mods: `getUnitById`, `getUnitAt`,
`getUnitAtXY`, `getUnitClass`, `getAllUnitsForPlayer`, `getUnitState`,
`setUnitState`, `updateUnit(s)`, `unitSetGroove`, `spawnUnitEffect`,
`deleteUnitEffect`, `spawnMapAnimation`, `getWeaponDamage`,
`getTargetsInRange`, `getTargetsInRangeAfterMove`, `areAllies`,
`getCurrentPlayerId`, `getMapTriggers`, `playMapSound`,
`playPositionlessSound`, `moveUnitToOverride`, `waitTime`,
`setFacingOverride`.

## From official material

- Campaign editor: missions pass results to later missions through flags
  that span the whole campaign; branching paths; conditions such as an
  S-rank to reveal a secret mission (Chucklefish blog, "Scripted events").
- Wargroove 2: official mod support with documentation on the wiki
  (Chucklefish announcement); Conquest mode carries damaged or lost units
  over between battles.
- Price: both games list at $19.99 on Steam (aggregators; check the store).

## From the official modding guide

Summary of "Guide:Modding" on the Wargroove Wiki (last edited 2024-03-13,
content under CC BY-NC-SA; pasted by the user because the work environment
cannot reach the site). Modding is labelled a **beta feature**; support is on
the official Discord. The guide covers **both Wargroove 1 and 2**, with
format differences; tools and config files must not be mixed between them.

- **Tools:** `wargroove_mod_packer.exe <path_to_mod>` in
  `steamapps/common/Wargroove(2)/modpacker_bin`, which also ships example
  mods and many of the game's original assets
  (`original_assets/config/...`, `original_assets/image/...`). Output goes to
  `%APPDATA%/Chucklefish/Wargroove(2)/mods`; errors are logged to
  `mod_packer_mod.txt`.
- **Mods are per map or campaign.** Maps can be packaged inside a mod (only
  if they use no other mod). Online play needs the same mod version on all
  sides.
- **New units: yes.** A new id in `unitClasses` adds a unit; it needs a unit
  class (cost, movement, moveRange, weapons, verbs, tags, battleUnitCount,
  sightRange in WG2), a unit config per faction (map and battle animation
  paths, portraits, sounds) and weapons. "New art and sounds can be used",
  so animation paths can presumably point to existing assets — to confirm
  on the first test. Existing classes can be modified (`modifyingUnitClasses`:
  add / replace / remove) or overwritten (same id).
- **Damage:** WG1 uses `tagDamage` per weapon; **WG2 reads a damage matrix
  `config/gameplay/units.csv`** (the game's own is in
  `original_assets/config/gameplay/units.csv`), with `tagDamageOverwrite` to
  reuse another unit's row.
- **Commanders, grooves, factions:** addable (commander, groove and verb
  configs; `factions.yaml`). WG2 adds **mini commanders** (strong units,
  commanders without a groove).
- **Removing** commanders, unit classes or factions through `manifest.yaml`.
- **Lua:** files under `lua/initialized/` get `init()` called, to register
  custom events and **replace game functions** (e.g. the combat system).
  Custom **trigger conditions and actions** for the editor (YAML + Lua +
  strings). **Verbs** (unit actions) with their **AI rules** in the same Lua
  file. Critical-hit ("passive") conditions per unit.
- **Cutscenes:** sound and visual effects, props, backgrounds, shouts,
  emotes, alternative runs and idles.
- **Strings:** one YAML per language, `fr-FR` supported.
- **Art:** animated assets are Aseprite files driven by **tags** (also usable
  from Lua); portraits and UI can be PNG. `.meta` files set the pivot.
  Palette-swapped assets (units and portraits: required) must use only the
  **Wargroove palette** (army colours, skin colours, fixed colours; full, no
  new colours). **The packer rejects any off-palette pixel and reports its
  colour and position.**
- **Terrain:** WG1 biomes and tiles can be added fairly simply. WG2 uses Wang
  tilesets that the guide calls fragile; it advises painting over the
  existing layouts rather than changing them.
- **Sound:** `.ogg`, 48 kHz; music and SFX through audio-event YAML.

## Wargroove 1 or 2 for this project

Same modding system and the same official guide for both; formats differ
and must not be mixed. Critics rated Wargroove 2 higher (Metacritic reviews
81–90, "a massive expansion" rather than a reinvention; criticisms: hand-
holding campaign, factions that play alike). Player opinion was not
measured.

| Point | Wargroove 1 | Wargroove 2 |
|---|---|---|
| Existing mod examples | More, older: the mods studied above date from 2019, before WG2 existed (UnitRanks very likely WG1) | Official support since the 2024 summer update; featured mods add factions |
| Damage tables | `tagDamage` per weapon | One matrix `units.csv`, easier to balance |
| Vision | — | `sightRange` per unit class |
| Elite units | — | Mini commanders |
| New terrain | Simple biome / tile configs | Wang tilesets, called fragile by the guide |
| Mod languages | + Italian, Korean | — |

Recommendation: close call. WG1 if the user prefers its feel, wants to
start from UnitRanks as is, or plans terrain art; WG2 for the damage matrix,
vision and mini commanders. The user should pick the game they enjoy
playing, since the mod lives inside it.

## Still to verify

1. **Lua access to data that persists between campaign missions** (needed
   for CO proficiency): not covered by the guide. The campaign editor has
   campaign-wide flags (official blog). Check the original Lua shipped in
   `modpacker_bin`, or ask on the Discord. Fallback: store proficiency
   tiers in campaign flags set by editor triggers.
2. **Spawning a unit from a verb** (engineer building a light camp): in
   Wargroove, structures are units (`isStructure`, `structure` damage tag),
   so a light camp can be a new structure unit class; the Lua function that
   spawns it is not confirmed yet (check the original verbs that recruit or
   reinforce).
3. **UI:** the guide covers icons, strings and portraits only. A
   post-battle report screen is unlikely; alternatives are cutscenes,
   events and verbs on the HQ (the Undo mod adds an HQ action).
4. Whether a new unit class can reuse an existing unit's animations by path.

## Sources

- https://github.com/Innocience/UnitRanks
- https://github.com/wargroove-modding/Pet-The-Dog
- https://github.com/wargroove-modding
- https://wargroove.com/scripted-events/
- https://x.com/ChucklefishLTD/status/1800539607194542230
- https://wargroovewiki.com/Guide:Modding (pasted by the user; CC BY-NC-SA)
