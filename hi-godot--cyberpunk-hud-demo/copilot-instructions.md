## cyberpunk-hud-demo

> Standalone Godot 4.6 project — a cyberpunk-styled HUD built entirely via MCP tools as a stress test for the [godot-ai](https://github.com/hi-godot/godot-ai) MCP server.

# Cyberpunk HUD Demo

Standalone Godot 4.6 project — a cyberpunk-styled HUD built entirely via MCP tools as a stress test for the [godot-ai](https://github.com/hi-godot/godot-ai) MCP server.

## What's here

- `cyberpunk_hud.tscn` — CanvasLayer-rooted HUD with health/shield bars, ability cooldowns, ammo counter, system log feed, and pause menu
- `themes/cyberpunk.tres` — dark navy/cyan/magenta theme with glowing panel shadows
- `autoload/game_state.gd` — demo state singleton (periodic damage, shield regen, cooldown ticks)
- `cyberpunk_hud.gd` — HUD controller wiring GameState signals to UI elements
- `hud_loader.gd` — runtime scene instancer (loads cyberpunk_hud.tscn into main.tscn)
- `main.tscn` — minimal 3D scene (camera + light + ground + HUD loader)

## MCP tools required

This project is designed to be edited via MCP tools from the godot-ai server. To use them:

1. Symlink or copy the plugin: `ln -s /Users/davidsarno/Documents/godot-ai/plugin/addons/godot_ai addons/godot_ai`
2. Enable the plugin in Project Settings > Plugins
3. Start the dev server: `cd ~/Documents/godot-ai && .venv/bin/python -m godot_ai --transport streamable-http --port 8000 --reload`
4. `session_activate` the cyberpunk-hud-demo session

## Input actions

- **Escape** — toggle pause menu
- **Q** — ability 1
- **E** (or Shift+E) — ability 2
- **R** — ability 3

## Animations on `/cyberpunk_hud/AP`

Active clips (referenced by cyberpunk_hud.gd):
- `hud_fade_in` — modulate fade on load (autoplay)
- `pause_slide_in` — overlay fade + menu slide
- `dmg_hit` — gentle panel shake + HDR red flash on damage
- `hp_glow` — low-health pulsing red glow (pingpong)

Stale clips (orphaned, can't delete without `animation_delete` tool):
- `damage_shake`, `dmg_flash`, `hp_low_pulse` — superseded by the above

## Origin

Built during a friction-logging exercise documented in `docs/friction-log-cyberpunk-hud.md`. The v2 visual polish plan is in `docs/prompt-hud-v2.md`.

---
> Source: [hi-godot/cyberpunk-hud-demo](https://github.com/hi-godot/cyberpunk-hud-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
