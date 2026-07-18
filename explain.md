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

## call_deferred — Deferred Function Calls
- **What:** Schedules a function call to run at the **end of the current frame**, after all physics processing is done
- **Why needed:** Godot throws "Can't change state while flushing queries" if you modify the scene tree (add/remove nodes) inside a `_physics_process` callback or collision signal. This happens because the physics engine is mid-calculation when you try to change things.
- **How:** `call_deferred("method_name")` — the method runs later, when it's safe
- **In our project:** `small_tree.gd` uses `call_deferred("add_log_scene")` when the tree is destroyed. The log can't be added during the physics callback that triggered the destruction, so it's deferred to end of frame.
- **C# analogy:** `Invoke(() => Method(), 0f)` in Unity, or `await Task.Yield()` then call the method. Same concept — don't modify the scene tree mid-physics.

## Spawn-on-Destroy Pattern
- **What:** When a destructible object dies, it spawns a dropped item at its position
- **How it works:**
  1. Tree script has `var log_scene = preload("res://scenes/objects/trees/log.tscn")`
  2. On `max_damage_reached` → `call_deferred("add_log_scene")` → `queue_free()`
  3. `add_log_scene()` instantiates the log, sets `global_position = global_position`, adds to `get_parent()` (the tree's parent node — the tilemap layer)
- **Why `get_parent()`:** The tree is being freed. You can't add a child to a node that's about to be destroyed. The parent (tilemap trees layer) persists, so the log goes there.
- **Why `call_deferred`:** You can't add nodes to the scene tree during a physics callback. Deferred ensures it happens at a safe time.
- **C# analogy:** Like spawning loot drops in Unity — `Instantiate(lootPrefab, transform.position, Quaternion.identity)` in `OnDeath()`, but deferred to end of frame.

## Collectable Pickup Pattern
- **What:** Dropped items auto-collect when the player walks over them
- **How:** CollectableComponent (Area2D) with `body_entered` signal → checks `if body is Player` → `get_parent().queue_free()`
- **Physics layers:** Collectable on layer 6 (mask=2 for Player), Player on layer 2. The collectable *detects* the player entering its area.
- **Why `get_parent().queue_free()` not `queue_free()`:** The CollectableComponent is a child of the Log node. Freeing the component leaves an orphaned log sprite. Freeing the parent removes the entire log.
