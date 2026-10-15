# AW2 Overhaul — Commander Wars mod

## Project

AW2 Overhaul is a gameplay mod for [Commander Wars](https://github.com/Robosturm/Commander_Wars),
an open-source Advance Wars clone. Commander Wars mods are folders of JavaScript
scripts (plus optional images/sounds) that the game loads at startup on top of
its own scripts.

Two repositories are usually open side by side:

- `AW2-overhaul/` — this mod. **The only repository we modify.**
- `Commander_Wars/` — the original game. **Read-only reference, never modify it.**
  Use it for the script API, and as a model (unit scripts, action scripts,
  bundled example mods).

## Ground rules

- **Never invent an engine function.** Every call on a C++ object (`unit.`,
  `map.`, `action.`, `campaign.`…) must exist in Commander_Wars: either a
  `Q_INVOKABLE` method or a `public slots:` method in the matching header under
  `Commander_Wars/game/` or `Commander_Wars/coreengine/`, or a function already
  defined in `Commander_Wars/resources/scripts/`. If something we need is
  missing, say so to the user instead of working around it silently.
- **The user tests everything in the game themselves.** There is no automated
  test suite and no way to run the game here. Each change must come with a
  short, concrete in-game test checklist.
- Documentation (this file, README) is written in English. Conversation with
  the user is in French.

## Vision

### Milestone 0 — validate the toolchain

A minimal mod that loads and changes one infantry stat (its cost). Status:
implemented in `scripts/units/infantry.js` (cost 1000 → 1500, test value),
waiting for the user's in-game test.

### Axes of the overhaul

1. **Persistent unit experience and progression.** As the CO advances through
   the campaign, they manage their units better: units earn XP, get promoted
   and gain perks. Build on the existing rank system if possible.
2. **New units, built from scratch** (units that do not exist in the game yet):
   - **Drones** — modern warfare.
   - **MANPADS** — infantry specialised in anti-air.
   - **Engineer** — builds *light camps*: deployable field bases giving
     resupply, deployment of light units and resource generation.
3. **New map(s)** using existing sprites, adding a varied new battlefield
   (the user wants level-design advice).
4. **Visual redesign** — extra combat animations and more detailed map
   sprites.
5. **Reputation / romance between characters** (stretch goal, potentially
   game-breaking) — a light RPG layer adding flavour outside the core loop.

A short campaign with its own story ties these together.

The grenade action from the first plan is **dropped**.

Design decisions, open questions and the technical approach per axis are in
`docs/design.md`; sprite formats are in `docs/sprites.md`. Keep both up to
date when the user decides something.

## Current status

- The user is installing the game on a computer and will test from there.
  Until the first in-game test passes, keep code changes small.
- The user is preparing sprites and assets with PixelLab and Retro
  Diffusion; check them against `docs/sprites.md`.
- The user has a fork of Commander Wars (the `Commander_Wars` repository in
  the session). It stays read-only: the mod must work with the official
  game. Any engine (C++) change needs the user's explicit decision.

## Repository layout

The repository root *is* the mod folder. It is installed by cloning it as
`<Commander Wars>/mods/aw2_overhaul` (the folder name is the mod's id).

```
mod.txt          mod metadata (read by the game)
scripts/         mirrors Commander_Wars/resources/scripts/
  units/         e.g. units/infantry.js patches INFANTRY
  actions/       new or patched actions (ACTION_*)
  ...
images/          (later) sprites, each folder with its res.xml
CLAUDE.md, README.md
```

## How Commander Wars loads a mod (verified in the source)

- `mod.txt`: `key=value` lines — `name`, `description`, `version`,
  `compatible_mods`, `incompatible_mods`, `required_mods` (`;`-separated mod
  paths such as `mods/awds_unit`), `cosmetic`, `tags`, optional `thumbnail`.
  Parsed in `Commander_Wars/coreengine/settings.cpp` (`Settings::getModInfos`).
  Each line is split on `=`, so never put a `=` inside a value.
- Each script file defines a global object named after the file in upper case
  (`units/infantry.js` → `INFANTRY`).
- Load order (`Commander_Wars/resource_management/ressourcemanagement.h`,
  `loadAll`): base scripts from `resources/scripts/<category>/` first, then the
  same category in every active mod. Active mods are sorted alphabetically,
  so the last one loaded wins.
- Therefore a mod file at the same path **patches** the existing object
  instead of replacing it: redefine only what changes, e.g.
  `INFANTRY.getBaseCost = function() { return 1500; };`
  (see `Commander_Wars/mods/awds_unit/scripts/units/infantry.js`).
- A new object (new unit, new action) is a full file following the base
  pattern: `var Constructor = function() { ... }; Constructor.prototype = UNIT;
  var MY_UNIT = new Constructor();`.
- Files whose name starts with `__` are loaded before the others, files
  starting with `___` after all the others; neither is registered as an object
  id (useful for shared helpers or patches that need everything loaded).

## Conventions

- Patch, don't copy: override only the functions or table entries we change.
  Keep the same relative path as the base script.
- New objects use an `AW2O_` prefix only if a name could clash with the base
  game; otherwise follow the base naming (`ACTION_GRENADE`, `ENGINEER`…).
  Decide once per object type and keep it.
- Every player-facing string goes through `qsTr("...")`.
- Comment non-obvious game rules, not the obvious code. Match the style of the
  base scripts (4-space indent, `this.fn = function(...)` inside constructors).
- Bump `version=` in `mod.txt` for every change the user will test
  (multiplayer compares mod versions).
- Watch for conflicts with bundled gameplay mods that patch the same objects
  (for instance `mods/awds_unit` also patches `INFANTRY.getBaseCost`). Test
  with other gameplay mods disabled; list known conflicts in
  `incompatible_mods` when relevant.

## Where to look in Commander_Wars

| Topic | Location |
|---|---|
| Unit scripts | `resources/scripts/units/` (`infantry.js`, `mech.js`, `ZCOUNIT_*.js`) |
| Unit base class / defaults | `resources/scripts/general/unit.js` |
| Unit API (C++) | `game/unit.h` |
| Actions | `resources/scripts/actions/` — `ACTION_FIRE.js` (combat, rank-up on kill), `ACTION_FLARE.js` (pick a target field in range), `ACTION_BUILD_TEMP_AIRPORT.js` (unit builds a temporary building), `ACTION_SUPPORTSINGLE_SUPPLY.js` (resupply) |
| Action base / API | `resources/scripts/general/action.js`, `game/gameaction.h` |
| Weapons and damage tables | `resources/scripts/weapons/` |
| Movement types | `resources/scripts/movementtables/` |
| Ranks | `resources/scripts/general/unitrankingsystem.js` (`UNITRANKINGSYSTEM`: names, icons, offensive/defensive bonus, `increaseRang`); `unit.getUnitRank()` / `unit.setUnitRank()` in `game/unit.h` |
| CO perks | `resources/scripts/co_perks/` (77 perks) |
| Buildings / terrain | `resources/scripts/building/`, `resources/scripts/terrain/` |
| Battle animations | `resources/scripts/battleanimations/` |
| Campaigns | `resources/scripts/general/BaseCampaign.js`, `maps/*.jsm` (campaign scripts), `maps/*.camp/` (campaign maps + per-map scripts) |
| Persistent data | `campaign.getVariables()` (kept across a campaign's maps), `map.getVariables()`, `unit.getVariables()` — API in `coreengine/scriptvariables.h` and `coreengine/scriptvariable.h` (`createVariable`, `writeDataInt32`, `readDataInt32`, …) |
| Dialogs / cutscenes | `GameAnimationFactory.createGameAnimationDialog` in `game/gameanimation/gameanimationfactory.h`; examples in `maps/*.camp/*.js` |
| Example mods | `mods/` — `awds_unit` (unit/weapon patches), `awdc_terrain` (small patch), `aw2_co` (CO + action patches), `aw_styled_icons` (images) |
| Mod loading | `coreengine/settings.cpp`, `coreengine/virtualpaths.cpp`, `resource_management/ressourcemanagement.h` |

## Testing (done by the user, in the game)

1. Clone or copy this repository to `<Commander Wars>/mods/aw2_overhaul`.
2. In the game options, open the mod list, enable **AW2 Overhaul**, restart.
3. Check the console (F1 by default) and the `console-*.log` file in the
   user folder for script errors.
4. Follow the checklist given with each change.
