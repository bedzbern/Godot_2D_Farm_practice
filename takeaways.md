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
