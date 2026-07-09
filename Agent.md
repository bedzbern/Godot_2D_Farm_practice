# Agent.md — Session Memory

## Project
Godot 2D Farm Game (practice project)

## Course
- Title: **How to Build a Complete 2D Farming Game** (8h, 25 episodes)
- Engine: **Godot 4.3**
- URL: https://www.youtube.com/watch?v=it0lsREGdmc

## Guidelines
- Keep notes SHORT — bullet points, no paragraphs
- Focus on actionable Godot patterns
- Always check Agent.md first in a new session to know where we left off
- **AI acts as a buddy** — proactively suggest improvements, best practices, optimizations, and point out things the user might miss. Don't wait to be asked.
- Saying **"push git"** auto-stages all, commits with auto-generated description, and pushes
- Saying **"UPDATE MY MD'S"** triggers full sync: user provides timestamp → check Godot project (scenes, scripts, assets) for current state → search YT transcript at that timestamp for context → update all MD files accordingly

### File Purposes
| File | Content |
|------|---------|
| **takeaways.md** | General Godot tips & reusable patterns (for future projects) — NOT project-specific settings |
| **journal.md** | Session log — what we did, specific settings/configs used in this project |
| **progress.md** | Topic index — searchable table of topics + timestamps per episode. For future reference ("what timestamp was that topic?"). Also has full episode chapter list |
| **Agent.md** | Session memory for the AI — current state, history, decisions, references |
| **explain.md** | Deep dives on Godot concepts explained in simple terms |

## Session History
| # | Date | What we did | Next step |
|---|------|-------------|-----------|
| 1 | 2026-07-09 | Setup — created note files, registered course | Start Episode 1 |
| 2 | 2026-07-09 | Started Ep 1 @ 5:31, renamed assets, cleaned duplicates, extracted YT transcript, added takeaways, restructured progress.md as topic index | Continue Ep 1 @ 20:08 — near end of Tutorial 2 |
| 3 | 2026-07-09 | Finished Tutorial 2 (Tilemap Layer), created test_scene_tilemap.tscn, added texture filter takeaway | Start Ep 3 — Player with State Machine (21:08) |

## Current State
- **Session ended.** Watched through Tutorial 2 @ t=1208 (20:08) — wrapping up nature tilemap & texture filter fix
- Tutorial 2 completed: water, grass, tilled dirt, nature tilemap layers + terrain sets/bit masks
- Project has `test_scene_tilemap.tscn` created
- Next session: **Ep 3 — Creating the player with a state machine (21:08)**
- All assets renamed to lowercase snake_case with updated .import paths
- YT transcript extracted and saved to desktop reference
- "UPDATE MY MD'S" & "push git" keyword conventions added

## Reference Files
- **YouTube Transcript**: `C:\Users\DNTS\Desktop\opencode_ref\yt_transcript_it0lsREGdmc.json`
  - Full transcript (10,360 entries) with timestamps in ms
  - When user gives a YouTube timestamp (`t=SECONDS`), search by `offset` field to find matching dialogue

## Tech / Decisions
- Godot version: 4.3
- Asset naming convention: lowercase snake_case (e.g. `wooden_house.png`)
- Key scenes, scripts, patterns used: [TBD as we build]
