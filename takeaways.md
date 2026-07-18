# Takeaways — Quick Reference

Short Godot tips, patterns, and things to remember.
Add entries as we learn.

## Test Scene Methodology
- Build each game component in its own test scene (isolation)
- Tilemap, player, NPCs, etc. each get a separate test scene
- When final level breaks, debug by checking the specific component's test scene
- Makes troubleshooting individual systems much easier

## Pixel Art — Texture Filter
- Go to Project → Project Settings → Rendering → Textures
- Set **Default Texture Filter** → **Nearest**
- Fixes blurred sprites in pixel art games

## Component Pattern — Data + Logic Split
- **HitComponent** = pure data (tool type + damage amount), sits on the attacker (player)
- **HurtComponent** = filter + signal, sits on the damageable object (tree), checks tool match
- **DamageComponent** = health tracking + threshold signal, sits on damageable object
- **Glue script** on the object connects the signals together
- This pattern is reusable: same components work for trees, rocks, chests, etc. — just change the tool type and max_damage

## Physics Layers for Tool System
- Layer 4 = Tool (HitComponent collision_layer)
- Layer 5 = Object (HurtComponent collision_layer)
- HitComponent: layer=8, mask=16 (detects objects)
- HurtComponent: layer=16, mask=8 (detects tools)
- Mirror setup ensures only tool↔object interactions trigger, not object↔object or tool↔tool

## call_deferred — Deferred Calls
- **Never modify the scene tree during a physics callback or collision signal**
- Use `call_deferred("method_name")` to schedule work for end of frame
- Common error: "Can't change state while flushing queries" — means you're adding/removing nodes mid-physics
- C# equivalent: `Invoke(() => ..., 0f)` in Unity

## Spawn-on-Destroy Pattern
- Object dies → `call_deferred("spawn_item")` → `queue_free()`
- Spawn method: instantiate scene, set `global_position`, add to `get_parent()` (not to self — you're being freed)
- `preload()` for scenes you always need, `load()` for ones you might not

## CollectableComponent Pattern
- Area2D + `body_entered` signal → check `if body is Player` → `get_parent().queue_free()`
- Free parent (the item node), not self (the component) — otherwise the sprite orphan persists
- collision_layer = Collectable layer, collision_mask = Player layer
