# AW2 Overhaul — Game design

Living document. Records the user's design decisions, the open questions, and
the technical approach checked against the Commander Wars source.
Values marked *(placeholder)* are starting points to tune after playtests.

## Axis 1 — Persistent experience and progression

### Decisions

- **Two layers of progression.**
  - **Unit XP (in battle):** a unit earns XP for every meaningful interaction
    with the map or the enemy — dealing damage, destroying a unit, surviving,
    capturing, etc. Each counts independently.
  - **CO XP (after battle):** at the end of a battle, the CO gains proficiency
    with each unit type, weighted by how efficiently those units were used.
    Example: four infantry that performed well → next battle, infantry costs
    less and gets damage and/or resistance bonuses.
- **Ranks:** keep the existing 4 ranks and bonuses for now
  (Soldier / Experienced / Veteran / Elite; offence +0/+5/+10/+20 %,
  defence +0/+0/+0/+20 %).
- **Feats of arms → passives:** extra passives unlocked by what units of a
  type actually did. Example: a unit type that captures a lot gains +25 %
  capture speed. Requires a persistent record of per-unit-type counters
  (captures, kills, damage dealt…).
- **Perk choice, roguelite style:** the player picks perks from a random
  selection, for replayability.
- **Death and merging:** a unit that dies loses its own XP; the CO still gets
  the unit-type XP for having used it. XP is kept when units are merged
  (joined).
- **AI:** the AI gains XP too, scaled by a difficulty coefficient (to tune).

### Open questions

1. Does the system run outside the campaign (skirmish / versus /
   multiplayer), or only in the campaign?
2. **Units do not carry over between maps** in Commander Wars: each map
   starts with its own units. Should individual unit XP last only for one
   battle (only the CO / unit-type layer persists), or do we build a roster
   that redeploys surviving veterans on the next map?
3. Definition of "efficient use" for the CO weighting (for example damage
   dealt in funds ÷ funds spent on that type, kills, survival rate).
4. Ranking threshold changes (the kill-based rank-up): to decide after
   playing the current version.
5. Scope of the feats record: one campaign only, or across all campaigns of
   the player?
6. When is the random perk choice offered (after each mission? at the start
   of the next one?), how many choices, and caps on cumulative bonuses
   (maximum discount, maximum damage bonus)?

### Technical approach (verified in the source)

- **CO perks are the backbone.** A perk is a script object
  (`resources/scripts/co_perks/`, base `CO_PERK` in
  `resources/scripts/general/co_perk.js`) and `co.addPerk(id)`,
  `co.removePerk(id)`, `co.getPerkList()` are callable from scripts
  (`game/co.h`). The engine queries every perk of a CO for hooks, among
  them: `getCostModifier`, `getOffensiveBonus`, `getDeffensiveBonus`,
  `getCaptureBonus`, `postBattleActions`, `buildedUnit`, `startOfTurn`
  (signatures in `resources/scripts/general/co.js`). So:
  - CO proficiency (discount, damage, resistance per unit type) = mod perks
    added to the player's CO at map start.
  - A hidden "tracker" perk can listen to `postBattleActions` (damage dealt)
    and other hooks to award unit XP. The exact list of event hooks to use
    must be confirmed one by one when implementing.
- **Persistence:**
  - Within a campaign: `campaign.getVariables()` keeps data across maps
    (used by the official campaigns, e.g. `maps/advance_wars_campaign.jsm`).
  - On a unit during a battle: `unit.getVariables()` (saved with the game,
    `game/unit.cpp` serialises it).
  - Across campaigns: `Userdata.getScriptVariableFile(filename)` exists
    (`coreengine/userdata.h`); its use from scripts must still be checked.
  - API: `createVariable`, `getVariable`, `writeDataInt32` / `readDataInt32`,
    `writeDataString`, list variants (`coreengine/scriptvariable(s).h`).
- **Ranks:** `UNITRANKINGSYSTEM` (`resources/scripts/general/unitrankingsystem.js`).
  Rank-up on kill happens in `ACTION_FIRE.performPostUnitAnimation`. The
  rank bonuses only apply when the game rule "ranking system" is on.
- **Not verified yet:** a script-driven selection screen for the perk choice
  (between missions or at map start).

## Axis 2 — New units

### Decisions

- **Drone:** one all-rounder unit to start with.
- **MANPADS:** anti-air infantry, attack range 2 (between infantry and
  artillery). Against infantry: like an anti-tank infantry unit (mech).
  Against vehicles: like basic infantry.
- **Engineer and light camp:**
  - 1 engineer = 1 light camp, then the engineer has no resources left until
    resupplied by an APC.
  - The engineer is **not** consumed when building.
  - Maximum 2 light camps per player.
  - A light camp can be **destroyed, not captured**.
  - Can be built anywhere except high mountains and water.
  - A light camp counts as **half a regular building** to start with
    (income, repair, defence…).
  - The camp provides resupply, deployment of light units and income.
- **AI:** must be able to use every new unit from the start; scenarios can
  restrict this later to match the difficulty.

### Open questions

1. **Drone:** air unit? Movement, fuel, vision, cost? Which units can hit it
   (anti-air, MANPADS, fighters)? Built at a factory or an airport?
2. **MANPADS:** damage against air units (its main role)? Minimum range 1
   or 2? Can it move and fire in the same turn (artillery cannot, infantry
   can)? Can it capture? Cost?
3. **Engineer:** does a regular city also refill its resource, or only an
   APC? Can it capture, fight?
4. **Light camp:**
   - "Half a building" in numbers (proposal: income 500 instead of 1000,
     repairs 1 HP instead of 2, defence 1 star) *(placeholder)*.
   - Which units can it deploy (infantry, mech, MANPADS, drone, recon…)?
   - How is it destroyed: attacked like a unit with HP, or something else?
   - Can a destroyed camp be rebuilt (the 2-camp limit counts live camps)?

### Technical approach (verified in the source)

- New units follow the base pattern (`resources/scripts/units/`); the
  `ZCOUNIT_*` units are good examples of non-standard units.
- Every weapon has a `damageTable` listing target unit ids: a new unit
  needs entries in every weapon that can hit it, and its own weapons need a
  table (see `resources/scripts/weapons/`).
- `ACTION_BUILD_TEMP_AIRPORT` (unit builds a temporary building) is the
  model for the light camp; `ACTION_SUPPORTSINGLE_SUPPLY` for APC resupply.
  Proposal: model the engineer's "resource" as a 1-shot ammo slot, so the
  existing APC resupply refills it.
- **Risk — AI and custom actions.** The AI (C++, `ai/`) knows special
  actions by name: `ACTION_FLARE`, `ACTION_EXPLODE`… are hard-coded in
  `ai/coreai.cpp`, and nothing in `ai/` references the temporary-airport
  construction. A new "build light camp" action would therefore most likely
  be ignored by the AI without changes to the engine (C++). Units that only
  use standard actions (fire, move, capture) should be handled by the AI
  normally; to confirm in game.

## Axis 3 — New map(s)

Built with the in-game map editor using existing sprites (the `.map` format
is binary and edited in the game). Scripted events go in the map's `.js`.
Level-design advice to be given when we start this axis.

## Axis 4 — Visual redesign

Extra combat animations and more detailed map sprites. Mostly art work; the
scripts live in `resources/scripts/battleanimations/`. See `docs/sprites.md`
for formats.

## Axis 5 — Reputation / romance (stretch)

Light RPG layer: affinity between characters, flavour outside the core loop.
Feasible in a light form during missions with campaign variables and the
dialogue system (`GameAnimationFactory.createGameAnimationDialog`). A screen
between missions is not verified.

## Short campaign

Ties the axes together, with its own story. Campaign structure:
`resources/scripts/general/BaseCampaign.js`, official examples in `maps/`.
