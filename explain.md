# Explain — Concepts I Wanted Clarified

Deep dives on specific Godot concepts, written in simple terms.
Add entries when you ask me to explain something.

## Test Scene Methodology
- **What:** Build every game component in its own isolated test scene instead of dumping everything into the main level
- **Why:** When the final level breaks, you can go back to the specific component's test scene to check if it works in isolation — narrows down where the bug is
- **Our project:** We have a `scenes/test/` folder. Currently have `test_scene_tilemap.tscn`. Future test scenes for player, NPCs, collectibles, etc.
- **Pattern:** One test scene per major system (tilemap, player, NPC, UI, etc.)
