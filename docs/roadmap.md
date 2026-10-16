# Roadmap

Planning for the Wargroove 2 version of the project. Estimates assume the
user works about **8 hours a week**, Claude writes most of the code, and the
user tests in game. Update the status column at each session.

Status values: `todo`, `doing`, `done`, `blocked`, `dropped`.

## Milestones

| # | Milestone | Target (weeks from start) | Definition of done |
|---|---|---|---|
| M0 | Ready to mod | 2 | Game bought and installed, example mod packaged and loaded in game |
| M1 | Risks cleared | 4 | The three technical spikes answered (yes / no / workaround), recorded in `docs/engines/wargroove.md` |
| M2 | Vertical slice | 9 | One mission playable start to end with the modern faction (placeholder art), unit XP and two dialogue scenes |
| M3 | Art pipeline proven | 9 (in parallel) | One modern unit made through the chosen art workflow, accepted by the mod packer, seen in game |
| M4 | Systems complete | 14 | CO proficiency, perks, engineer + light camp, drone, MANPADS working |
| M5 | Campaign complete | 20 | 3 to 5 missions, both endings, reputation scenes |
| M6 | Release 1.0 | 24 | Balanced with outside playtesters, French and English text, mod package shared |

Total ≈ 180–230 hours, about 6 months. Phases 3 (art) and 2–4 (code) run in
parallel.

## Phase 0 — Setup (M0)

| Task | Owner | Est. | Status |
|---|---|---|---|
| Buy and install Wargroove 2 | User | 0.5 h | todo |
| Play the start of the campaign (lore, feel) | User | 4–6 h | todo |
| List the folders of `modpacker_bin` and send them | User | 0.5 h | todo |
| Package an example mod, load it in a map | User + Claude | 1–2 h | todo |

## Phase 1 — Technical spikes (M1)

Each spike is a small throwaway test that answers one risky question.

| Spike | Question | Fallback if "no" | Est. | Status |
|---|---|---|---|---|
| A — New unit | Can a new unit class reuse the archer's animations by path? | Overwrite an existing class instead | 3 h | todo |
| B — Persistence | Can Lua read data kept between campaign missions? | Proficiency tiers stored in campaign flags set by editor triggers | 4 h | todo |
| C — Spawn | Can a verb spawn a structure unit (light camp)? | Engineer converts an existing structure, or camps pre-placed on maps | 3 h | todo |

## Phase 2 — Vertical slice (M2)

| Task | Est. | Status |
|---|---|---|
| Modern faction skeleton: faction config, 3 unit classes (rifleman, MANPADS, drone) with placeholder art | 6 h | todo |
| Damage matrix rows for the new units (`units.csv`) | 3 h | todo |
| Unit XP and ranks, starting from the UnitRanks approach | 8 h | todo |
| Mission 1 map in the editor | 6 h | todo |
| Two dialogue scenes (intro, end) | 4 h | todo |
| Playtest and fixes | 5 h | todo |

## Phase 3 — Art pipeline (M3, then ongoing)

| Task | Est. | Status |
|---|---|---|
| Choose the workflow (see `docs/art-workflow.md`) | 2 h | todo |
| First unit end to end: map + battle animations, palette, pivots, packer OK | 10–20 h | todo |
| Remaining roster (5–6 units total), portraits for original characters | 30–60 h | todo |

## Phase 4 — Systems (M4)

| Task | Est. | Status |
|---|---|---|
| CO proficiency after each mission (per unit type) | 10 h | todo |
| Perk choice (HQ verb or cutscene, depending on spike B) | 8 h | todo |
| Engineer + light camp (depending on spike C) | 10 h | todo |
| Drone and MANPADS final stats and AI behaviour | 6 h | todo |
| Balance pass | 6 h | todo |

## Phase 5 — Campaign (M5)

| Task | Est. | Status |
|---|---|---|
| Characters and story outline final (`docs/story.md`) | 6 h | todo |
| Missions 2 to 4 (or 5): maps, triggers, dialogues | 30–40 h | todo |
| Two endings and the choice | 6 h | todo |
| Reputation / romance scenes (light) | 8 h | todo |

## Phase 6 — Release (M6)

| Task | Est. | Status |
|---|---|---|
| Outside playtest (2–3 people) and fixes | 8 h | todo |
| Text in French and English | 4 h | todo |
| Packaging, description, screenshots, sharing | 4 h | todo |

## Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Cross-mission data not reachable from Lua | CO proficiency coarser | Spike B early; flag-based fallback |
| Generated art not good enough | Faction looks off | Prove the pipeline on one unit before scaling; keep placeholders playable |
| Time and motivation | Project stalls | Small milestones; the vertical slice is shareable on its own |
| Modding is a beta feature | Tool bugs | Ask on the official Discord; keep backups of saves and maps |

## Session routine

1. Pick one task from the current phase.
2. Work, then test in game with the checklist Claude provides.
3. Update the status here and record new ideas in `docs/ideas.md`.
