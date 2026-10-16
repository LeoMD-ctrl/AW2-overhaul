# Modding vault kit (Obsidian)

A starter vault for **all** modding projects, not only AW2 Overhaul. Copy
this folder somewhere outside the project repositories, open it in
Obsidian ("Open folder as vault"), and keep one vault for every game.

Notes and templates are in French (the user's working language).

## Layout

| Folder | Holds | Reusable across games? |
|---|---|---|
| `00 Inbox` | Raw ideas, pasted chat excerpts, to sort later | — |
| `10 Projets` | One folder per mod project: links to its repository, status | No (project-specific) |
| `20 Moteurs` | One note per game / engine: how its modding works, limits, sources | Yes |
| `30 Patterns` | Game design patterns (progression, economy, AI scoring, level design) | Yes |
| `40 Pipelines` | Art, audio and packaging workflows | Yes |
| `50 Prompts` | Prompt library (image generators, writing assistants) | Yes |
| `60 Outils` | Tools: Aseprite, Blender, Git, Lua, YAML… | Yes |
| `90 Templates` | Note templates | Yes |

## Plugins

- Core: Templates (folder `90 Templates`), Canvas, Graph view, Properties.
- Community (optional): Dataview (queries in `Accueil.md`), Templater,
  Obsidian Git (sync the vault to a private GitHub repository).

## With Claude

Project repositories keep the detailed, versioned docs (`docs/` in each
repo). The vault keeps what can be reused. At the end of a session, ask
Claude for a "vault extract": the reusable lessons, patterns and prompts
from the session, formatted with these templates.
