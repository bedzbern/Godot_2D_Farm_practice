# Explain — Concepts I Wanted Clarified

Deep dives on specific Godot concepts, written in simple terms.
Add entries when you ask me to explain something.

## Test Scene Methodology
- **What:** Build every game component in its own isolated test scene instead of dumping everything into the main level
- **Why:** When the final level breaks, you can go back to the specific component's test scene to check if it works in isolation — narrows down where the bug is
- **Our project:** We have a `scenes/test/` folder. Currently have `test_scene_tilemap.tscn`. Future test scenes for player, NPCs, collectibles, etc.
- **Pattern:** One test scene per major system (tilemap, player, NPC, UI, etc.)

## Component Pattern — Hit → Hurt → Damage → Destroy
- **What:** A signal-driven system where behavior is split into small reusable components instead of one big script
- **How it works:**
  1. **HitComponent** (on player) = "I hit with THIS tool for THIS damage"
  2. **HurtComponent** (on tree) = "I only care about THIS tool type" → emits `hurt` signal
  3. **DamageComponent** (on tree) = "I have X health, add damage, emit when dead"
  4. **Glue script** (small_tree.gd) = "Connect hurt→apply_damage→queue_free"
- **Why split it:** Each component is reusable. HitComponent works for axe/pickaxe/watering can. HurtComponent works for trees/rocks/chests. DamageComponent works for anything with health. You just swap the exported tool type in the Inspector.
- **Physics layers:** Tool (4) and Object (5) are mirror layers — HitComponent detects Object layer, HurtComponent detects Tool layer. This prevents false positives (two trees overlapping won't trigger each other).
- **C# analogy:** Same as Unity's component pattern — `MonoBehaviour` components on a GameObject that communicate via events. HitComponent = data-only component, HurtComponent = event filter, DamageComponent = health system, small_tree.gd = controller that wires them up.
