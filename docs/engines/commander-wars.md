# Engine notes: Commander Wars

Prototype target explored first (October 2026). Status: **on hold** — the
user found the game's PC playability, interface and polish too weak (see
`candidates.md`). Everything below was verified in the Commander Wars
source and stays valid if the project comes back to this engine.

Reference state of the prototype: commit `a1118ce` on branch
`claude/gifted-galileo-p254kj` (mod version 0.2.0, infantry cost patched to
1500, never tested in game).

## Prototype files in this repository

The repository root is currently laid out as a Commander Wars mod
(installed by cloning it as `<Commander Wars>/mods/aw2_overhaul`):

```
mod.txt                  mod metadata
scripts/units/infantry.js  milestone 0 patch (cost 1000 → 1500)
art/palettes/            colour palette and code mapping (CW colour tables)
art/templates/           frame grids at CW sizes
```

## Rules when working on this engine

- `Commander_Wars/` (the user's fork, open in the session) is a read-only
  reference: the mod must work with the official game. Any C++ change needs
  the user's explicit decision.
- Never invent an engine function: every call on a C++ object (`unit.`,
  `map.`, `action.`, `campaign.`…) must exist as a `Q_INVOKABLE` or
  `public slots:` method in `Commander_Wars/game/` or
  `Commander_Wars/coreengine/`, or as a function in
  `Commander_Wars/resources/scripts/`.

## How Commander Wars loads a mod

- `mod.txt`: `key=value` lines — `name`, `description`, `version`,
  `compatible_mods`, `incompatible_mods`, `required_mods` (`;`-separated mod
  paths such as `mods/awds_unit`), `cosmetic`, `tags`, optional `thumbnail`.
  Parsed in `coreengine/settings.cpp` (`Settings::getModInfos`). Each line is
  split on `=`, so never put a `=` inside a value.
- Each script file defines a global object named after the file in upper
  case (`units/infantry.js` → `INFANTRY`).
- Load order (`resource_management/ressourcemanagement.h`, `loadAll`): base
  scripts from `resources/scripts/<category>/` first, then the same category
  in every active mod. Active mods are sorted alphabetically; the last one
  loaded wins.
- A mod file at the same path **patches** the existing object: redefine only
  what changes (see `mods/awds_unit/scripts/units/infantry.js`).
- A new object is a full file: `var Constructor = function() { ... };
  Constructor.prototype = UNIT; var MY_UNIT = new Constructor();`.
- Files starting with `__` load before the others, `___` after all others;
  neither is registered as an object id.

## Conventions

- Patch, don't copy; keep the base script's relative path.
- Player-facing strings go through `qsTr("...")`.
- Match the base scripts' style (4-space indent, `this.fn = function(...)`).
- Bump `version=` in `mod.txt` for every change to test.
- Bundled gameplay mods can patch the same objects (`mods/awds_unit` patches
  `INFANTRY.getBaseCost`): test with them disabled.

## Where to look in Commander_Wars

| Topic | Location |
|---|---|
| Unit scripts | `resources/scripts/units/` (`infantry.js`, `mech.js`, `ZCOUNIT_*.js`) |
| Unit base class / API | `resources/scripts/general/unit.js`, `game/unit.h` |
| Actions | `resources/scripts/actions/` — `ACTION_FIRE.js`, `ACTION_FLARE.js`, `ACTION_BUILD_TEMP_AIRPORT.js`, `ACTION_SUPPORTSINGLE_SUPPLY.js`, `ACTION_BUILD_UNITS.js` (menu input) |
| Action base / API | `resources/scripts/general/action.js`, `game/gameaction.h` |
| Weapons and damage tables | `resources/scripts/weapons/` |
| Ranks | `resources/scripts/general/unitrankingsystem.js` |
| CO perks (hooks for XP, costs, bonuses) | `resources/scripts/co_perks/`, `resources/scripts/general/co_perk.js`, `game/co.h` |
| Buildings / terrain | `resources/scripts/building/` (`temporary_airport.js`), `resources/scripts/terrain/` |
| Battle animations | `resources/scripts/battleanimations/`, `game/gameanimation/battleanimationsprite.h` |
| AI script hooks | `resources/aidata/normal/normalai.js`, `ai/coreai_predefinedai.cpp` |
| UI in XML + JS | `resources/ui/`, `ui_reader/uifactory.*`; `map.showXmlFileDialog(...)` |
| Campaigns | `resources/scripts/general/BaseCampaign.js`, `maps/*.jsm`, `maps/*.camp/` |
| Persistent data | `campaign.getVariables()`, `map.getVariables()`, `unit.getVariables()` (`coreengine/scriptvariable(s).h`) |
| Mod loading | `coreengine/settings.cpp`, `coreengine/virtualpaths.cpp`, `resource_management/ressourcemanagement.h` |

Per-feature technical approaches are in `docs/design.md` (sections
"Technical approach") and sprite formats in `docs/sprites.md`; both describe
this engine.

## Testing

1. Copy the repository to `<Commander Wars>/mods/aw2_overhaul`.
2. Options → mod list → enable **AW2 Overhaul** alone, restart.
3. Check the console (F1) and `console-*.log` in the user folder.
