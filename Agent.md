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
| **scripts_notes.md** | Code explanations for every .gd script — GDScript + C# comparisons for reference |
| **godot_errors.md** | Comprehensive error catalog + debugging tool reference — stack traces, null safety, input troubleshooting, profiling |

### Keywords
- **"push git"** — auto-stage all, commit with auto-generated description, push
- **"UPDATE MY MD'S"** — full sync: check project state, search YT transcript at given timestamp, update all MD files
- **"explain the code"** — scan recently modified .gd files, check scripts_notes.md for what's already explained, explain NEW/changed scripts with C# comparisons

## Session History
| # | Date | What we did | Next step |
|---|------|-------------|-----------|
| 1 | 2026-07-09 | Setup — created note files, registered course | Start Episode 1 |
| 2 | 2026-07-09 | Started Ep 1 @ 5:31, renamed assets, cleaned duplicates, extracted YT transcript, added takeaways, restructured progress.md as topic index | Continue Ep 1 @ 20:08 — near end of Tutorial 2 |
| 3 | 2026-07-09 | Finished Tutorial 2 (Tilemap Layer), created test_scene_tilemap.tscn, added texture filter takeaway | Start Ep 3 — Player with State Machine (21:08) |
| 4 | 2026-07-09 | Started Ep 2 @ 21:08, moved to new PC, re-fetched YT transcript, set up player with 20 animations & collision, added to test scene — up to 29:52 | Continue Ep 2 — State machine & keyboard inputs |
| 5 | 2026-07-09 | Fixed idle_state.gd input/animation bugs, created scripts_notes.md & godot_errors.md, idle state working with debugger — up to 40:55 | Continue — refactor input into game_input_events.gd, create walk state |
| 6 | 2026-07-10 | Finished Ep 2 (up to 53:05 / t=3185). Created GameInputEvents, WalkState, Player class. Idle↔Walk working. | Start Ep 3 — Tool states (chop, till, water) |

## Current State
- **Completed Episode 2** (up to t=3185 / 53:05). Full player state machine working: Idle ↔ Walk with all 4-direction animations.
- Files created/updated this session:
  - `scripts/input_events.gd` — static helper `GameInputEvents` (`movement_input()`, `is_movement_input()`)
  - `scenes/characters/player/walk_state.gd` — walk movement at speed=50, idle transition on release
  - `scenes/characters/player/player.gd` — `class_name Player`, `extends CharacterBody2D`, `player_direction` var
  - `idle_state.gd` — refactored to use `player.player_direction` + `GameInputEvents`, emits "Walk" transition
  - `player.tscn` — Player script, Walk state node, all node paths set
- Next up: **Start Episode 3 — Tool states (chop, till, water) at 53:47**
- Project structure: 6 GDScripts, 4 scenes
- `scripts_notes.md` needs entries for input_events.gd, walk_state.gd, player.gd, updated idle_state.gd
- YT transcript: NOT available on this PC (was on Administrator PC). Consider re-downloading. Transcript ref path in Agent.md is stale.
- All keyword conventions active: "UPDATE MY MD'S", "push git", "explain the code"

## Reference Files
- **YouTube Transcript**: `C:\Users\Administrator\Desktop\opencode_ref\yt_transcript_it0lsREGdmc.json`
  - Full transcript (10,360 entries) with timestamps in ms
  - When user gives a YouTube timestamp (`t=SECONDS`), search by `offset` field to find matching dialogue

## Tech / Decisions
- Godot version: 4.3
- Asset naming convention: lowercase snake_case (e.g. `wooden_house.png`)
- Key scenes, scripts, patterns used: [TBD as we build]
