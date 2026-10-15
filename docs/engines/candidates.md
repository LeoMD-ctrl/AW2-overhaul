# Engine candidates

The game design (`docs/design.md`) is engine-independent; the engine is not
chosen yet. The user's criteria, after trying Commander Wars:

1. **Polish and feel** comparable to the GBA Advance Wars games.
2. **Good playability on PC** (controls, interface).
3. Room for the design: persistent XP, new units, buildable camps,
   campaign with story, light RPG layer.

Last reviewed: 2026-10-15. Facts below come from the sources linked; items
marked *to verify* were not confirmed.

## Commander Wars (open source, Advance Wars clone)

- Fit for the design: excellent — every feature was checked against the
  source as feasible as a script mod (see `commander-wars.md`).
- Polish / PC playability: **rejected by the user** (interface, controls,
  overall polish). These are engine-level (C++ and XML UI) and cannot be
  fixed by a mod.
- Status: on hold.

## Wargroove / Wargroove 2 (commercial, PC, Advance Wars-like)

- Polish: a finished commercial game with its own universe (no Nintendo IP
  issue).
- Official mod support: YAML configuration, Lua scripts, art and sound
  files, Modpacker tool; Aseprite is the recommended animation tool.
  Campaign editor with branching campaigns, flags and counters in triggers.
  Wargroove 2 also announced official mod support.
- Precedents: community mods add unit actions in Lua (e.g. a "pet the dog"
  action in the wargroove-modding GitHub organisation); players rebuilt the
  Advance Wars campaign in the editor; Wargroove 2's Conquest mode carries
  damaged or lost units over between battles.
- Verified in published mods (details in `wargroove.md`): new unit actions
  in Lua that the AI can use, per-unit data, combat logic overrides, new
  editor trigger actions; an existing mod (UnitRanks) already implements
  unit XP and ranks.
- *To verify:* brand-new unit classes and structures, Lua access to data
  that persists between campaign missions (needed for CO XP), how far the
  UI can be changed.

## Advance Wars 2 ROM hack (GBA, played through an emulator)

- Polish: the original game's.
- Decompilation `Eebit/aw2bhr`: 5 commits, 13 stars, mostly assembly,
  completion not stated, needs the user's own ROM for every build. Not a
  usable base today.
- Without a decompilation: stats, damage tables, maps and graphics (GBA
  limits: 16-colour palettes, small tiles) are realistic; new systems (XP,
  buildable camps, new unit types, reputation) mean writing ARM assembly in
  a binary not designed for it — months of reverse engineering.
- Realistic scope: rebalance, new maps, graphics, and reusing existing unit
  slots (e.g. a unit restyled and rebalanced as MANPADS).
- Legal: distribute only patches (IPS/BPS), never the ROM.

## AI-assisted modding tools (supplied by the user)

- **REA — Reverse Engineer Anything** (`morluto/rea`, npm `rea-agents`,
  MIT): an MCP server that lets an AI agent analyse binaries through an
  existing engine (Ghidra, IDA or Hopper), plus apps, APKs, firmware,
  websites. No GBA mention; with Ghidra (which handles ARM) it could help
  understand a GBA ROM, but it does not remove the cost of patching it.
  The star count shown on the page looked unusually high; download only
  from the official source, as its README itself warns.
- **universal-modder** (`rehan-remade/universal-modder`, MIT): skills, a CLI
  and a knowledge base for AI agents (Claude Code included) to mod PC games
  the user owns; Windows first; asset generation through fal (paid API) or
  local ComfyUI. No GBA support. Could help on a PC target such as
  Wargroove.

## Sources

- https://github.com/Eebit/aw2bhr
- https://github.com/morluto/rea
- https://github.com/rehan-remade/universal-modder
- https://wargroovewiki.com/Guide:Modding
- https://github.com/wargroove-modding
- https://www.pcgamesn.com/wargroove/wargroove-mod-support
- https://wargroove.com/spring-update-v1-2-6-mod-support-new-merch-japanese-physical/
- https://www.pcgamesn.com/wargroove/wargroove-advance-wars
