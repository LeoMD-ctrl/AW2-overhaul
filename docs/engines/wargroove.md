# Engine notes: Wargroove

Candidate engine, reviewed 2026-10-15. The user owns neither Wargroove nor
Wargroove 2. The official modding guide (wargroovewiki.com) could not be
read from the work environment (host blocked by its network policy), so the
facts below come from the source of published mods and from official blog
posts.

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

## Still to verify (needs the official guide)

1. Can a mod add **brand-new unit classes** and **structures**, not only
   modify existing ones? (Needed for drone, MANPADS, engineer, light camp.)
2. Can **Lua** read and write the campaign-wide flags or other data that
   persists between missions? (Needed for CO proficiency.)
3. How far can the **UI** be changed (post-battle report, perk choice)?
4. Differences between Wargroove 1 and 2 mod support.

## Sources

- https://github.com/Innocience/UnitRanks
- https://github.com/wargroove-modding/Pet-The-Dog
- https://github.com/wargroove-modding
- https://wargroove.com/scripted-events/
- https://x.com/ChucklefishLTD/status/1800539607194542230
- https://wargroovewiki.com/Guide:Modding (not reachable from the work environment)
