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
| 7 | 2026-07-10 | Finished Ep 3 & 4 (up to 1:06:15 / t=3975). Created all 3 tool states (chopping, tilling, watering), DataTypes enum, tool transitions from idle. Fixed if/elif bug in chopping. | Start Ep 5 — Creating houses |
| 8 | 2026-07-10 | Finished part of Ep 5 (up to 1:18:06 / t=4686). Created house tileset, large_house.tscn scene, SceneCollectionSource in game_tile_set, test scene with houses layer. Adjusted player collision. | Continue Ep 5 or start Ep 6 (choppable trees) |
| 9 | 2026-07-10 | Finished Ep 5 (up to 1:28:14 / t=5294). Created InteractableComponent, Door scene with open/close, collision layers setup. Debug prints commented out. | Start Ep 6 — Creating choppable trees |
| 10 | 2026-07-12 | Started Ep 6 — Choppable Trees (t=5499 / 1:31:39). Watching intro. | Continue Ep 6 — HitComponent, HurtComponent, DamageComponent, SmallTree, Log |
| 11 | 2026-07-15 | Finished Ep 6 up to t=6616 (1:50:16). Created HitComponent, HurtComponent, DamageComponent, SmallTree, test_scene_object_trees. Updated ChoppingState with hit detection. Tree chopping working (3 hits to destroy). | Continue Ep 6 — Log scene, CollectableComponent (from ~1:50:16) |
| 12 | 2026-07-18 | Finished Ep 6 (up to t=6951 / 1:55:51). Created Log scene (Sprite2D + CollectableComponent), CollectableComponent (Area2D), updated SmallTree to spawn log on destroy, created LargeTree (max_damage=5), created small_house.tscn & medium_house.tscn. Fixed transcript path (E:→D:). | Start Ep 7 — Tree shake vertex shader (1:56:10) |

## Current State
- **Finished Episode 6** at t=6951 (1:55:51) — full tree system complete
- Complete tree destruction flow: Player chops → HitComponent hits → HurtComponent filters tool → DamageComponent tracks health → on destroy → Log spawned via `call_deferred` → CollectableComponent picks up on player contact
- Two tree variants: SmallTree (3 hp, 16x32 atlas) and LargeTree (5 hp, 32x32 atlas, higher y-sort position)
- Three house variants: large_house, medium_house, small_house — all using same house_tile_set.tres
- **Next: Episode 7** — Tree shake vertex shader (1:56:10)
- Project structure: 17 GDScripts, 21 scenes, 2 TileSets
- Physics layers: Ground (1), Player (2), Interactable (3), Tool (4), Object (5)
- Test scenes: `test_scene_tilemap`, `test_scene_player`, `test_scene_house_tilemap`, `test_scene_object_trees`
- All keyword conventions active: "UPDATE MY MD'S", "push git", "explain the code"

## Reference Files
- **YouTube Transcript**: `D:\GOdot_Youtube_farming_transcript\yt_transcript_it0lsREGdmc.json` (10,360 entries, timestamps in ms)
  - Full transcript (10,360 entries) with timestamps in ms
  - When user gives a YouTube timestamp (`t=SECONDS`), search by `offset` field to find matching dialogue

## Tech / Decisions
- Godot version: 4.3
- Asset naming convention: lowercase snake_case (e.g. `wooden_house.png`)
- Key scenes, scripts, patterns used: [TBD as we build]
