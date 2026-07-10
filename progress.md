# Progress — Course Tracking / Topic Index

Course: **How to Build a Complete 2D Farming Game** (Godot 4.3)
Link: https://www.youtube.com/watch?v=it0lsREGdmc

## Ep 1 — Project Setup & Tilemap Layers (0:00 – 21:08)

| Topic | Timestamp |
|-------|-----------|
| Intro & project setup | 1:35 |
| Creating test scene tilemap | 6:31 |
| Test scene methodology (isolated components) | 6:58 |
| Viewport / display settings for pixel art | 10:33 |
| Water tilemap layer | 12:20 |
| Grass tilemap layer + terrain sets / bit masks | 14:30 |
| Tilled dirt tilemap layer | 17:00 |
| Nature tilemap layer (decorations) | 19:00 |
| Texture filter → nearest (pixel art fix) | 20:30 |

*✅ Completed*

## Ep 2 — Player with State Machine (21:08 – 53:47)

| Topic | Timestamp |
|-------|-----------|
| Intro to player & state machine | 21:08 |
| Creating test scene player (duplicate test_scene_default) | 21:46 |
| Creating player scene (CharacterBody2D) | 23:00 |
| Adding AnimatedSprite2D with sprite frames | 23:55 |
| Setting up 20 animations (idle/walk/chop/till/water × 4 dirs) | 25:00 |
| Setting FPS to 3 for all animations | 26:45 |
| Adding CollisionShape2D (circle) | 29:10 |
| Adding player to test scene | 29:52 |

| Debugging with breakpoints (F10/F12) | 39:50 |
| Refactoring input code into reusable class (game_input_events.gd) | 40:55 |

| Creating GameInputEvents static helper class (input_events.gd) | ~42:00 |
| Creating Player class (player.gd) with player_direction var | ~44:00 |
| Creating WalkState (walk_state.gd) with movement & animations | ~46:00 |
| Connecting state transitions: Idle ↔ Walk (on_enter/on_exit) | ~48:00 |
| Testing player movement with full state machine | ~50:00 |

*✅ Completed*

## Ep 3 & 4 — Tool States: Chopping, Tilling, Watering (53:47 – 1:06:18)

| Topic | Timestamp |
|-------|-----------|
| Creating DataTypes Tools enum | ~55:00 |
| Creating GameInputEvents.use_tool() | ~56:00 |
| Adding current_tool to Player | ~57:00 |
| Creating ChoppingState with direction animation | ~58:00 |
| Creating TillingState | ~1:00:00 |
| Creating WateringState | ~1:01:00 |
| Connecting tool transitions in IdleState | ~1:02:00 |
| Setting up tool state nodes in player.tscn | ~1:03:00 |
| Animation loop settings for tool actions (loop off) | ~1:04:00 |
| Testing all 3 tool states | ~1:05:00 |

*✅ Completed*

## Ep 5 — Creating Houses (1:06:18 – 1:31:12)

| Topic | Timestamp |
|-------|-----------|
| Creating house tileset (walls + furniture) | ~1:07:00 |
| Setting up physics collision on house tiles | ~1:08:00 |
| Building large_house.tscn with TileMapLayers | ~1:10:00 |
| Adding house as SceneCollectionSource in game_tile_set | ~1:14:00 |
| Creating test_scene_house_tilemap with Houses layer | ~1:15:00 |
| Adjusting player collision radius (9→4) & position | ~1:17:00 |
| Creating InteractableComponent (Area2D, signals) | ~1:19:00 |
| Creating Door scene with open/close animations | ~1:22:00 |
| Adding door SceneCollectionSource to house_tile_set | ~1:25:00 |
| Setting up collision layers (Player=2, Interactable=3) | ~1:26:00 |
| Commenting out debug prints in state machine | ~1:27:00 |

*✅ Completed*

## All 25 Episode Chapters (Full Video)
| # | Title | Timestamp |
|---|-------|-----------|
| 1 | How to setup your project | 1:35 |
| 2 | Using Tilemap Layer node to design game tiles | 5:50 |
| 3 | Creating the player with a state machine | 21:08 |
| 4 | Creating tool states for your player | 53:47 |
| 5 | Creating houses using tilesets and tilemap layers | 1:06:18 |
| 6 | Creating choppable trees | 1:31:12 |
| 7 | Tree shake vertex shader | 1:56:10 |
| 8 | Creating mineable rocks | 2:04:06 |
| 9 | Y-sorting for characters & objects | 2:16:13 |
| 10 | Chicken NPC with navigation agents | 2:26:42 |
| 11 | Cow NPC with reusable components | 3:02:13 |
| 12 | Navigation regions, agents & avoidance | 3:09:10 |
| 13 | UI — Tools panel | 3:19:25 |
| 14 | Collectables with reusable components | 3:45:27 |
| 15 | UI — Inventory panel | 3:55:59 |
| 16 | Day and Night component | 4:15:19 |
| 17 | Farming crops (plants & corn) | 4:46:27 |
| 18 | Tilling land (dynamic tiles) | 5:16:25 |
| 19 | Save system (data components & resources) | 5:37:08 |
| 20 | Interactive guide NPC with dialogue | 6:01:55 |
| 21 | Custom dialogue balloons & scripts | 6:19:47 |
| 22 | Interactable chest with inventory | 6:53:21 |
| 23 | First level with all components | 7:23:44 |
| 24 | Main menu UI | 7:43:53 |
| 25 | Audio & SFX with audio bus | 8:15:08 |
