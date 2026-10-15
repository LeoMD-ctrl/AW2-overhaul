# AW2 Overhaul

## Project

A fan project: an Advance Wars-style tactics experience with deeper
progression, new modern units, new maps and a short story campaign.

The **game design is the core of this repository** and does not depend on
an engine. The engine (or game to mod) is **not chosen yet**: see
`docs/engines/candidates.md`. A first prototype targeted Commander Wars and
is on hold (`docs/engines/commander-wars.md`).

## Vision — five axes

1. **Persistent experience and progression.** Units earn XP in battle; after
   each battle the CO gains proficiency with the unit types used, weighted
   by efficiency (cost discounts, damage / resistance bonuses), plus feats
   of arms and roguelite perk choices. Campaign only.
2. **New units built from scratch:** drone (air all-rounder, fragile),
   MANPADS (anti-air infantry), engineer building *light camps* (field
   bases: resupply, basic infantry production, income).
3. **New maps** using existing sprites, with level-design care.
4. **Visual redesign:** more detailed sprites and richer combat animations.
5. **Reputation / romance between characters** (stretch goal): a light RPG
   layer outside the core loop.

A short campaign with its own story ties the axes together.

## Where things are

| Path | Content |
|---|---|
| `docs/design.md` | Design decisions, proposed XP model, open questions. Its "Technical approach" sections describe Commander Wars. |
| `docs/engines/candidates.md` | Engine comparison against the user's criteria, with sources. |
| `docs/engines/commander-wars.md` | Everything verified about Commander Wars, and the prototype's state. |
| `docs/sprites.md` | Sprite specification for Commander Wars (sizes, colour coding, battle scenes). |
| `art/` | Drawing palette and frame templates (Commander Wars sizes). |
| `mod.txt`, `scripts/` | Commander Wars prototype (milestone 0, untested). |

## Ground rules

- **Design first, engine second.** Record every design decision in
  `docs/design.md`, in engine-neutral terms. Engine-specific details go in
  `docs/engines/<engine>.md`.
- **Never invent an API or a tool capability.** Any claim about what an
  engine, game or tool can do must come from its source code or official
  documentation, cited. If something needed is missing or unverified, say
  so to the user.
- **The user tests everything themselves** (no test environment here).
  Each change comes with a short, concrete test checklist.
- **The user's criteria for an engine:** polish and feel close to the GBA
  games, good PC playability, room for the design. Commander Wars failed
  the first two.
- **Legal care:** no copyrighted game files in the repository (no ROMs, no
  extracted assets); ROM hacks ship as patches only. Reference renders of
  another game's art stay out of the repository.
- Repositories open in a session other than this one (such as the
  `Commander_Wars` fork) are read-only references unless the user decides
  otherwise.
- Documentation is in English; conversation with the user is in French.

## Current status

- Engine choice pending. Next step: verify what Wargroove's mod support can
  do for the design (see the *to verify* items in `candidates.md`).
- The user produces sprites with Retro Diffusion, PixelLab and Aseprite.
  Sizes and palettes depend on the engine, so large art batches wait for
  the engine choice.
