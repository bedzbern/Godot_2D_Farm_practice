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
- Next: state machine & keyboard inputs
