# Journal — Session Log

Short dated entries of what we did.

## 2026-07-09
- Session 1: Setup — created note files, registered course

## 2026-07-09 (Session 2)
- Started Episode 1 @ 5:31
- Renamed all assets to lowercase snake_case convention
- Cleaned up duplicate space-named files
- Updated .import source_file paths to match
- Committed & pushed "add game assets" (63 files)
- Added "UPDATE MY MD'S" keyword convention to Agent.md
- Extracted full YT transcript (10,360 entries) → `Desktop\opencode_ref\yt_transcript_it0lsREGdmc.json`
- Committed & pushed "rename assets to lowercase snake_case, add yt transcript ref"
- Added "push git" auto-commit convention
- Added test scene methodology takeaway
- Watched Tutorial 2 (Tilemap Layer) through to end @ 20:08
- Learned: terrain sets / bit masks, tilemap layers (water, grass, tilled dirt, nature), viewport pixel art settings, texture filter → nearest
- Created `test_scene_tilemap.tscn`
- Restructured progress.md as topic index
- Added texture filter takeaway for pixel art

## 2026-07-09 (Session 3)
- Started Episode 2 @ 21:08 (moved to new PC, re-fetched YT transcript)
- Watched through ~29:52 (collision shape added, player in test scene)
- Created `test_scene_player.tscn` (duplicated from default)
- Created `player.tscn` (CharacterBody2D) with AnimatedSprite2D
- Set up 20 animations: idle/walk/chop/till/water × 4 directions, all at 3 FPS
- Added CollisionShape2D (circle) to player
- Created state machine scripts (`node_state.gd`, `node_state_machine.gd`)
- Created `idle_state.gd` with input-based direction/animation
- Fixed input detection bugs (is_action_just_pressed → is_action_pressed, walk_front → walk_up)
- Fixed animation name mismatches (idle_up → idle_front, front → idle_front)
- Created `scripts_notes.md` with C#-compared code explanations
- Created `godot_errors.md` comprehensive debugging reference
- Watched through ~40:55 — idle state working, about to refactor input into reusable class
- Next: create game_input_events.gd and walk state

## 2026-07-10 (Session 6)
- Watched Episode 2 from 40:55 to 53:05 (t=3185) — near end of Episode 2
- Created `scripts/input_events.gd` (`class_name GameInputEvents`) — static input helper with `movement_input()` and `is_movement_input()`
- Created `scenes/characters/player/walk_state.gd` — walk state with movement (speed=50), direction tracking, idle transition on release
- Created `scenes/characters/player/player.gd` (`class_name Player`) — `extends CharacterBody2D` with `player_direction` var
- Updated `idle_state.gd` — now uses `player.player_direction` instead of direct Input calls, transitions to "Walk" on input
- Updated `player.tscn` — Player script attached, Walk state node added to StateMachine, all node paths configured
- Next: Start Episode 3 — Tool states (chop, till, water)
