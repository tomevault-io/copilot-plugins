## love2d-pocket-bomber-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pocket Bomber is a Bomberman-style grid-based action game built with LÖVE (Love2D) 11.5 in Lua. It targets both desktop (keyboard) and iOS (touch). All visuals use shape-based rendering — no external sprite assets.

## Running the Game

```bash
love .
```

Requires LÖVE 11.5 installed. There is no build step, test suite, or linter — the game runs directly from source.

## Architecture

### Game Loop & State Machine

`main.lua` wires LÖVE callbacks (`load`, `update`, `draw`, `resize`, input events) to three systems: Camera, Input/Touch, and StateMachine. The state machine (`src/state_machine.lua`) manages four states — **menu**, **playing**, **paused**, **gameover** — each with `enter(params)`, `update(dt)`, `draw()`, `exit()`, and input handler methods.

### Camera & Coordinate System

The camera (`src/camera.lua`) scales a logical 960×540 space to the actual window, stretching to fill. Tile size and grid offsets are calculated dynamically from the viewport. All touch/mouse coordinates pass through `Camera.toGame()` before reaching game logic. The top 80 logical pixels are reserved for HUD.

### Grid System

The grid (`src/grid.lua`) is a 13×9 2D array of tile types: `EMPTY` (0), `WALL_PERMANENT` (1), `WALL_BREAKABLE` (2). Coordinate conversions between grid `(col, row)` and world `(x, y)` live in `src/utils.lua`. Collision uses per-corner hitbox checks against grid tiles.

### Entity Model

Entities are plain Lua tables managed in lists on `src/globals.lua` (`G.bombs`, `G.explosions`, `G.enemies`, `G.particles`). The player is a singleton. Each entity type has its own module:

- **Player** (`src/player.lua`) — Cardinal movement at 180px/s, cornering assist (auto-align within 12px), bomb pass rule (can walk through just-placed bomb until leaving its tile), death animation sequence.
- **Enemy** (`src/enemy.lua`) — Three types (basic/fast/tank) with simple roaming AI: continue forward, randomly turn perpendicular, reverse if stuck for 0.5s. Avoids bomb tiles.
- **Bomb** (`src/bomb.lua`) — 2.0s fuse with accelerating blink rate. Returns true on detonation to trigger explosion spawn.
- **Explosion** (`src/explosion.lua`) — Cross-shaped propagation up to 2-3 tiles per direction. Stops at permanent walls, destroys breakable walls. Has a 0.12s grace window before dealing damage (fairness buffer). Total duration 0.5s.
- **Particle** (`src/particle.lua`) — Float text (score popups) and debris (block destruction).

### Level Data

`src/level_data.lua` defines 5 handcrafted levels. Each specifies a grid layout string, breakable wall positions, enemy spawn list, player start position, blast radius, and time limit (120-200s). `src/level.lua` handles loading and progression.

### Input Abstraction

`src/input.lua` normalizes keyboard input into `G.input` (dx/dy direction, bombPressed, pausePressed). `src/touch_controls.lua` provides a floating virtual joystick (left half) and bomb/pause buttons (right side) on mobile. Platform detection chooses which to display.

### Persistence

`src/save.lua` reads/writes high scores via LÖVE's filesystem (identity: "pocket-bomber").

## Key Design Details

- **Globals pattern**: `src/globals.lua` exports a single table `G` holding all constants, shared state, and entity containers. Modules `require("src.globals")` to access it.
- **No external dependencies**: Pure LÖVE APIs only.
- **Modules disabled in `conf.lua`**: joystick, physics, video, thread.
- **dt capped at 0.1s** in the update loop to prevent large frame skips.
- **Color palette**: `src/colors.lua` converts hex to LÖVE RGB (0-1 range).

## Design Documents

- `docs/gdd.md` — Game Design Document with full mechanics specification
- `docs/tdd.md` — Technical Design Document with architecture pseudocode and iOS build details

---
> Source: [chongdashu/love2d-pocket-bomber-game](https://github.com/chongdashu/love2d-pocket-bomber-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
