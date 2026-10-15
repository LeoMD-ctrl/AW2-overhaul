# AW2 Overhaul — Game design

Living document. Records the user's design decisions, the open questions, and
the technical approach checked against the Commander Wars source.
Values marked *(placeholder)* are starting points to tune after playtests.

## Axis 1 — Persistent experience and progression

### Decisions

- **Campaign only.** The persistent CO progression runs only in the
  campaign, not in skirmish, versus or multiplayer.
- **Two layers of progression.**
  - **Unit XP (in battle):** a unit earns XP for every meaningful interaction
    with the map or the enemy — dealing damage, destroying a unit, surviving,
    capturing, etc. Each counts independently.
  - **CO XP (after battle):** at the end of a battle, the CO gains proficiency
    with each unit type, weighted by how efficiently those units were used.
    Example: four infantry that performed well → next battle, infantry costs
    less and gets damage and/or resistance bonuses.
- **Unit XP lasts one battle (option a).** Units do not carry over between
  maps in Commander Wars; only the CO / unit-type layer persists. This puts
  the emphasis on CO perks.
  - *Future idea, not planned:* a veteran roster that redeploys surviving
    units on the next map (option b). The user likes it; it is heavier to
    build but feasible in the mod: `map.spawnUnit(x, y, unitID, owner, ...)`
    is script-callable (`game/gamemap.h`).
- **Ranks:** keep the existing 4 ranks and bonuses for now
  (Soldier / Experienced / Veteran / Elite; offence +0/+5/+10/+20 %,
  defence +0/+0/+0/+20 %). Threshold changes to decide after playing.
- **Feats of arms → passives, per campaign.** Extra passives unlocked by
  what units of a type did during the current campaign. Example: a unit type
  that captures a lot gains faster capture. Stored in campaign variables.
- **Perk choice, roguelite style:** the player picks perks from a random
  selection.
  - **Unit perks:** offered during the battle through a **"Promotion"
    unit action**, available once the unit has been promoted (from the end
    of the turn of its promotion). The action opens a menu of random perks.
  - **CO perks:** offered at the end of the battle, in a post-battle report
    that also grants the CO's XP.
- **Death and merging:** a unit that dies loses its own XP; the CO still gets
  the unit-type XP for having used it. XP is kept when units are merged
  (joined).
- **AI:** the AI gains XP too, scaled by a difficulty coefficient (to tune).

### Proposed model (to validate, all numbers are placeholders)

Principles: every coefficient lives in **one tuning table** (a single script
file), every number is logged at the end of a battle so playtests produce
data, and every bonus has a cap.

**Unit XP — events**

| Event | XP | Notes |
|---|---|---|
| Damage dealt | 1 per HP removed × value ratio | value ratio = target cost ÷ own cost, clamped 0.5–2: hitting a stronger unit pays more |
| Counter-attack damage | 50 % of the above | rewards holding ground without making defence the best farm |
| Destroying a unit | 10 + target cost ÷ 1000 | |
| Capture progress | 2 per capture point step, +10 on completion | |
| Surviving an attack | 3 | only when the unit was attacked, so parked units earn nothing |
| Support | 5 per action | APC resupply, engineer builds a camp, etc. |

Anti-farming: diminishing returns on the same target in a turn, cap of XP per
unit per turn *(placeholder 40)*.

Rank thresholds *(placeholder)*: 30 / 80 / 160 XP. On a promotion, the unit
gets its rank bonus and a unit-perk choice at the end of the turn.

**CO XP — efficiency per unit type (post-battle report)**

For each unit type T used during the battle (dead units included):

- `invested(T)` = funds spent on T (plus the value of pre-deployed units of T).
- `generated(T)` = value of the damage dealt by T (HP removed × target cost
  ÷ 10) + value of buildings captured + value of support actions.
- `efficiency(T)` = `generated(T) ÷ invested(T)`, clamped 0.5–2.
- `coXP(T)` = (sum of the unit XP of T) × `efficiency(T)` × outcome
  (victory 1.0, defeat 0.5) × difficulty coefficient.

Measuring efficiency as a ratio (value created ÷ value invested) means
spamming a type does not automatically pay: four well-used infantry beat
ten wasted ones.

Proficiency levels per unit type *(placeholder 0–5)*, each level giving for
example −3 % cost and +2 % offence/defence, capped at −15 % cost and
+10 % offence / defence. CO perks offered in the report come on top.

**Feats of arms (per campaign, per unit type)**

Counters: captures, kills, HP dealt, HP received and survived, support
actions, camps built. Thresholds unlock passives. Example: 15 captures →
faster capture. Note: the engine's capture bonus is additive (capture rate =
unit HP + bonus, `getCaptureBonus`), so "+25 %" translates to about +2 or +3
for a full-HP unit.

### Open questions

1. Validate or adjust the proposed model above (events, ratio, caps).
2. How many perks offered per choice (3?), and is the list shared between
   unit types or specific to each type?

### Technical approach (verified in the source)

- **CO perks are the backbone.** A perk is a script object
  (`resources/scripts/co_perks/`, base `CO_PERK` in
  `resources/scripts/general/co_perk.js`) and `co.addPerk(id)`,
  `co.removePerk(id)`, `co.getPerkList()` are callable from scripts
  (`game/co.h`). The engine queries every perk of a CO for hooks, among
  them: `getCostModifier`, `getOffensiveBonus`, `getDeffensiveBonus`,
  `getCaptureBonus`, `postBattleActions`, `buildedUnit`, `startOfTurn`,
  `getAdditionalBuildingActions` (signatures in
  `resources/scripts/general/co.js`). So:
  - CO proficiency (discount, damage, resistance per unit type) = mod perks
    added to the player's CO at map start.
  - A hidden "tracker" perk can listen to `postBattleActions` (damage dealt)
    and other hooks to award unit XP. The exact list of event hooks to use
    must be confirmed one by one when implementing.
- **Choice menus:** an action can show a menu, like the factory's build menu
  (`getStepInputType` returns `"MENU"`, `getStepData` fills it with
  `data.addData(...)`, see `ACTION_BUILD_UNITS.js`). A "Promotion" unit
  action listing random perks is therefore feasible.
- **Custom windows:** a script can open a window described in XML + JS with
  `map.showXmlFileDialog("ui/....xml")` (used by `ACTION_OPTIONS.js`;
  widgets: Label, Button, Checkbox, Panel, Spinbox, Textbox, Icon, Slider,
  Box… see `resources/ui/example.xml` and `ui_reader/uifactory.h`). Map
  scripts get a `victory(team, map)` hook (`resources/scripts/general/basegamescript.js`).
  A post-battle report window at victory is therefore a candidate; whether
  it shows before the game leaves the map must be tested. The campaign menu
  also loads `ui/campaignmenu.xml` from mods, so a progression panel there
  is possible.
- **Persistence:**
  - Within a campaign: `campaign.getVariables()` keeps data across maps
    (used by the official campaigns, e.g. `maps/advance_wars_campaign.jsm`).
  - On a unit during a battle: `unit.getVariables()` (saved with the game,
    `game/unit.cpp` serialises it).
  - API: `createVariable`, `getVariable`, `writeDataInt32` / `readDataInt32`,
    `writeDataString`, list variants (`coreengine/scriptvariable(s).h`).
- **Ranks:** `UNITRANKINGSYSTEM` (`resources/scripts/general/unitrankingsystem.js`).
  Rank-up on kill happens in `ACTION_FIRE.performPostUnitAnimation`. The
  rank bonuses only apply when the game rule "ranking system" is on.

## Axis 2 — New units

### Decisions

- **Drone:** one all-rounder, **built at the factory only**. An air unit
  following the air-unit rules, but vulnerable to basic infantry.
  **Weaker than helicopters, more mobile, more fragile.** Starts with the
  recon's characteristics (base game `recon.js`: cost 4000, movement 8,
  vision 5, fuel 80, machine gun with 10 ammo, range 1) with an air
  movement type.
- **MANPADS:** anti-air infantry. Moves and captures like infantry; fires
  **like the artillery: range 1–2, cannot move and fire in the same turn**.
  - Against infantry: like the anti-tank infantry (mech).
  - Against vehicles: like basic infantry.
  - Against air units: on the bazooka-vs-armour scale (numbers below).
- **Engineer and light camp:**
  - 1 engineer = 1 light camp, then the engineer has no resources left until
    resupplied, **only by a factory or an APC**.
  - The engineer is **not** consumed when building.
  - Maximum 2 light camps per player.
  - Can be built anywhere except high mountains and water.
  - A light camp counts as **half a regular building**: income 500 instead
    of 1000, repairs 1 HP instead of 2, defence 1 star.
  - Produces **basic infantry only**.
  - It is **destroyed by capturing it**: 10 capture points instead of 20 for
    a regular building; completing the capture removes the camp.
- **AI:** must be able to use every new unit from the start; scenarios can
  restrict this later to match the difficulty.

### Damage numbers (validated, to tune after playtests)

Reference values from the base game: bazooka vs APC 85, recon 75, light
tank 55, neotank 25, heavy tank 10; infantry machine gun vs transport copter
30, attack copter 8.

- **MANPADS vs air:** transport copter and duster 85, attack copter 75,
  drone 75, fighter and bomber 55, stealth bomber 25.
- **Infantry machine gun vs drone:** 45.
- Fragility of the drone: every weapon that can hit air units needs a drone
  entry above its transport-copter value (to define when the drone is built).

### Open questions

1. Drone: which weapon does it use (the recon's machine gun as is, or a
   drone-specific one), and which base units can hit it besides infantry
   and anti-air?

### Technical approach (verified in the source)

- New units follow the base pattern (`resources/scripts/units/`); the
  `ZCOUNIT_*` units are good examples of non-standard units.
- Every weapon has a `damageTable` listing target unit ids: a new unit
  needs entries in every weapon that can hit it, and its own weapons need a
  table (see `resources/scripts/weapons/`).
- `ACTION_BUILD_TEMP_AIRPORT` (unit builds a temporary building) is the
  model for the light camp; `ACTION_SUPPORTSINGLE_SUPPLY` for APC resupply.
- The temporary airport script defines `getBaseIncome`,
  `getConstructionList`, `getDefense`, `getRepairTypes`
  (`resources/scripts/building/temporary_airport.js`), and
  `BUILDING.repairUnit` takes the HP to repair as a parameter: income 500,
  infantry-only production, 1 star and 1 HP repair are all script settings.
- Engineer charge: store it in `unit.getVariables()` rather than in an ammo
  slot, because every repairing building refills ammo
  (`BUILDING.repairUnit` calls `unit.refill`); the factory and the APC
  supply action are then patched to restore the charge.
- Capture to destroy: `ACTION_CAPTURE.performPostAnimation` uses a local
  `maxCapturePoints = 20`; the mod can patch it to use 10 for a light camp
  and remove the camp instead of changing its owner.
- **AI and the engineer.** The C++ AI does not know a new "build camp"
  action by itself, but it asks scripts first: for every unit in the
  default AI mode, `CoreAI::processPredefinedGenericScripted`
  (`ai/coreai_predefinedai.cpp`) calls
  `GenericPredefinedUnitMapScriptBehaviour(ai, action, unit, enemies,
  enemyBuildings, map)` on the map script and on the AI's script object
  (`NORMALAI` in `resources/aidata/normal/normalai.js`, loaded with mod
  paths by `resource_management/gamemanager.cpp`). If it returns true, the
  AI performs the action the script prepared (`action.setActionID`,
  `action.setMovepath`, `ai.createTargetedPfs` for path finding).
  So the mod can give AI engineers a real routine — choose a spot, move,
  build — with no engine change. Equivalent objects exist for the other AI
  types (`aidata/very_easy/`, `aidata/heavy/`).
  Fallback if needed: a hidden perk on AI COs using `startOfTurn` to build
  where the engineer stands (`terrain.loadBuilding`, `unit.setHasMoved`).
  - To confirm in game: the AI buys infantry from a camp and captures
    enemy camps (both go through generic AI code).

## Mod or fork: feasibility summary

Status of each idea from the discussions, checked against the source.
"Mod" = scripts, XML UI and images only, works with the official game.

| Idea | Mod | Fork (C++) needed? |
|---|---|---|
| Unit XP, ranks, unit perks via a "Promotion" menu | Yes | No |
| CO progression per unit type (cost, offence, defence) | Yes, via CO perks | No |
| Feats of arms per campaign | Yes, campaign variables | No |
| Post-battle report window and CO perk choice | Likely (XML window + `victory` hook), to test | Only if the window cannot show at victory |
| Veteran roster across maps (option b) | Yes (`map.spawnUnit`), heavier | No |
| Drone, MANPADS, engineer, light camp | Yes | No |
| AI using engineers and camps | Yes (AI script hook) | Only for a deeper AI |
| New maps, campaign, dialogues | Yes (editor + scripts) | No |
| Reputation / romance during missions | Yes (variables, dialogues, XML windows) | No |
| Panels added to main / campaign / in-game menus | Yes (XML UI from mods) | No |
| Brand-new full-screen menus (e.g. a camp / barracks screen between missions) | No | Yes |
| More detailed sprites, longer battle animations | Yes (images + animation scripts) | No |
| New kinds of visual effects, shaders, a new battle-scene layout | No | Yes |
| Larger map tiles / different tile size | No (fixed 32 px, `GameMap::defaultImageSize`) | Yes, risky |
| Changing core rules hard-coded in C++ (damage formula, fog, turn flow) | Partly (many go through script hooks) | Yes for the rest |

## Axis 3 — New map(s)

Built with the in-game map editor using existing sprites (the `.map` format
is binary and edited in the game). Scripted events go in the map's `.js`.
Level-design advice to be given when we start this axis.

## Axis 4 — Visual redesign

Extra combat animations and more detailed map sprites. Mostly art work; the
scripts live in `resources/scripts/battleanimations/`. See `docs/sprites.md`
for formats and what the engine allows (higher-resolution sprites).

## Axis 5 — Reputation / romance (stretch)

Light RPG layer: affinity between characters, flavour outside the core loop.
Feasible in a light form during missions with campaign variables and the
dialogue system (`GameAnimationFactory.createGameAnimationDialog`). A screen
between missions is not verified.

## Short campaign

Ties the axes together, with its own story. Campaign structure:
`resources/scripts/general/BaseCampaign.js`, official examples in `maps/`.
