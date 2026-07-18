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

## 2026-07-10 (Session 7)
- Watched Episode 3 & 4 from 53:47 to 1:06:15 (t=3975)
- Created `scripts/globals/data_types.gd` — Tools enum (None, AxeWood, TillGround, WaterCrops, PlantCorn, PlantTomato)
- Moved/renamed `scripts/input_events.gd` → `scripts/game_input_events.gd`, added `use_tool()` method
- Updated `player.gd` — added `@export var current_tool: DataTypes.Tools`
- Created `scenes/characters/player/chopping_state.gd` — plays correct chopping animation by direction, auto-returns to Idle
  - Fixed `if`/`elif` bug where non-down directions all played chopping_front
- Created `scenes/characters/player/tilling_state.gd` — same pattern as chopping
- Created `scenes/characters/player/watering_state.gd` — same pattern as chopping
- Updated `idle_state.gd` — added tool transitions: Chopping (AxeWood), Tilling (TillGround), Watering (WaterCrops)
- Updated `player.tscn` — added Chopping, Tilling, Watering state nodes to StateMachine; set `current_tool = 2` (AxeWood)
- Changed all tool animation loops from 1 → 0 (play once, don't loop)
- Updated `test_scene_player.tscn` — instance changes
- Next: Start Episode 5 — Creating houses (1:06:18)

## 2026-07-10 (Session 8)
- Watched Episode 5 from 1:06:18 to 1:18:06 (t=4686) — Creating houses
- Created `tileset/house_tile_set.tres` — new TileSet with wooden house walls + furniture textures, physics collision on walls/furniture
- Created `scenes/Houses/large_house.tscn` — house scene with Floor, Walls, Furniture TileMapLayers
- Updated `tileset/game_tile_set.tres` — added `large_house.tscn` as SceneCollectionSource for tilemap placement
- Created `scenes/test/test_scene_house_tilemap.tscn` — test scene with game tilemap + Houses layer (instanced house) + player (current_tool = WaterCrops)
- Updated `player.tscn` — collision radius 9.0→4.0, collision position (0,-8)→(0,-4)
- Next: Continue Ep 5 or start Ep 6 (choppable trees @ 1:31:12)

## 2026-07-10 (Session 9)
- Watched Episode 5 from 1:18:06 to 1:28:14 (t=5294) — Interactable components & doors
- Created `scenes/components/interactable_component.gd` — reusable `InteractableComponent` (Area2D) with `interactable_activated`/`deactivated` signals
- Created `scenes/components/interactable_component.tscn` — Area2D with collision_layer=4, collision_mask=2
- Created `scenes/Houses/door.gd` — Door (StaticBody2D) with open/close animations, connects to InteractableComponent signals
- Created `scenes/Houses/door.tscn` — Door scene with AnimatedSprite2D (4-frame open/close at 5 FPS), RectCollisionShapes
- Updated `house_tile_set.tres` — added `door.tscn` as SceneCollectionSource
- Updated `player.tscn` — added `collision_layer = 2` (Player physics layer)
- Updated `project.godot` — added physics layers: Player (2), Interactable (3)
- Commented out debug prints in `node_state_machine.gd`
- Next: Episode 6 — Creating choppable trees (1:31:12)

## 2026-07-12 (Session 10)
- Started Episode 6 — Choppable Trees (t=5499 / 1:31:39)
- Watching intro — will create: HitComponent, HurtComponent, DamageComponent, SmallTree, Log, CollectableComponent
- Player chopping infrastructure already exists (chopping_state.gd, AxeWood tool)
- Need to create tree object with health/damage system and log drops

## 2026-07-15 (Session 11)
- Watched Episode 6 from 1:31:39 to 1:50:16 (t=6616)
- Created `scenes/components/hit_component.gd` + `.tscn` — Area2D, exports `current_tool` and `hit_damage`, collision_layer=8 (Tool), collision_mask=16 (Object)
- Created `scenes/components/hurt_component.gd` + `.tscn` — Area2D, exports `tool` type, emits `hurt` signal if tool matches, collision_layer=16 (Object), collision_mask=8 (Tool)
- Created `scenes/components/damage_component.gd` + `.tscn` — Node2D, exports `max_damage`/`current_damage`, `apply_damage()` with clamp, emits `max_damage_reached`
- Created `scenes/objects/trees/small_tree.gd` + `.tscn` — Sprite2D with atlas texture (basic_grass_biom_things.png), StaticBody2D for collision, HurtComponent + DamageComponent (max_damage=3), script connects hurt→damage→queue_free
- Updated `scenes/characters/player/chopping_state.gd` — added `hit_component_collision_shape` export, positions shape per direction on enter, disables on exit
- Updated `scenes/characters/player/player.tscn` — added HitComponent instance with CollisionShape2D (circle, radius=3)
- Updated `project.godot` — added physics layers: Tool (4), Object (5)
- Created `scenes/test/test_scene_object_trees.tscn` — test scene with tilemap, trees layer, player (current_tool=AxeWood)
- Tested: 3 axe hits destroys tree ✅
- Next: Create Log scene + CollectableComponent (from ~1:50:16)

## 2026-07-18 (Session 12)
- Watched Episode 6 from 1:50:16 to 1:55:51 (t=6951) — finished Ep 6
- Created `scenes/components/collectible_component.gd` + `.tscn` — Area2D, `body_entered` signal, checks `body is Player`, `get_parent().queue_free()`, collision_layer=32, collision_mask=2
- Created `scenes/objects/trees/log.tscn` — Sprite2D (atlas region 80,32 16x16), CollectableComponent (collectable_name="log"), CollisionShape2D (circle radius=8)
- Updated `scenes/objects/trees/small_tree.gd` — added `log_scene = preload(...)`, `add_log_scene()` method called via `call_deferred()` on max damage, spawns log at tree position on parent
- Created `scenes/objects/trees/large_tree.gd` + `.tscn` — same pattern as SmallTree, max_damage=5, atlas region (16,0, 32x32), RectangleShape2D hurt collision (10x24), position=(0,-15)
- Created `scenes/Houses/medium_house.tscn` — same tileset (house_tile_set.tres), different tile layout
- Created `scenes/Houses/small_house.tscn` — same tileset, smallest variant
- Fixed transcript path in Agent.md and progress.md (E: → D:)
- Pulled latest git, repo was up to date
- Next: Episode 7 — Tree shake vertex shader (1:56:10)
