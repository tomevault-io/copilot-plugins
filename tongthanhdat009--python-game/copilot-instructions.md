## python-game

> This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

A 2D platformer game built with Pygame. Features a player character with double-jump, wall-slide, dash, and skill abilities. Includes multiple enemy types (standard, special healing, boss), a tilemap editor, and a full menu system.

## Running the Game

```bash
# Launch the game
python test.py

# Launch the tilemap editor
python Editor.py
```

No build step required - pure Python + Pygame.

## Architecture

### Entry Points

- **`test.py`** - Main game (`Test` class). Manages game loop, menu system, pause screen, tutorial, and end-game screen via state-based method calls (`menu()` -> `run()` -> `pause()`/`endGame()`).
- **`Editor.py`** - Standalone tilemap editor (`Editor` class). Supports tile placement/removal, autotiling, on-grid/off-grid placement, and JSON map saving.

### Module Structure (`script/`)

All modules use `script.xxx` import pattern:

- **`script/utils.py`** - Image loading (`load_image`, `load_images`) and frame-based `animation` class. Base path: `data/images/`.
- **`script/entities.py`** - All physics entities. `PhysicsEntity` base class provides gravity, AABB collision with tilemap. `Player` extends it with double-jump, dash, wall-slide, skill. `Enemy`, `Spec_Enemy`, `Boss` extend it with patrol + shooting AI. All entity classes reference `self.game` for asset/audio access.
- **`script/tilemap.py`** - `Tilemap` class. Loads/saves JSON maps, provides physics collision rects, autotiling algorithm, and camera-offset rendering.
- **`script/bullet.py`** - `Bullet` class handles both enemy projectiles and player skills. Movement, collision, lifetime, hit detection.
- **`script/particles.py`** - `particle` class for visual effects. Plays animation once, self-terminates.
- **`script/spark.py`** - `Spark` class for diamond-shaped spark effects on hits/shots.
- **`script/spaceships.py`** - Parallax background ships/clouds with depth-based rendering.
- **`script/button.py`** - `Button` class for menu UI with hover + click sound feedback.

### Asset Structure (`data/`)

- **`data/images/`** - All visual assets. Entity sprites organized by animation state (idle/run/jump/hurt per entity type). Menu images with _1 (normal) and _2 (hover) variants. Tileset directories for grass, stone, industry, power station tiles.
- **`data/sfx/`** - Sound effects (wav/mp3).
- **`data/maps/`** - JSON map files (0.json-3.json) created by Editor.py.
- **`data/user.json`** - User progress (current level).
- **`data/font/`** - Custom pixel font.

### `pix2d/`

Raw pixel art source frames before integration into `data/images/`.

### Data Directory

- **`build/`** and **`dist/`** - PyInstaller output. Generated, do not edit.

## Key Conventions

- Path separator: uses `//` (Windows double-backslash) throughout. Fix to use `os.path.join` or `/` for cross-platform compatibility.
- Internal resolution: 320x240, scaled 2x to 640x480 window.
- Entity removal during iteration uses `.copy()` pattern.
- Class naming: lowercase convention (`animation`, `particle`, `spaceship`) - inconsistent with Python PEP 8.
- Enemy classes share nearly identical `update()` patterns with only different parameters - candidate for refactoring.
- Damage/death logic is duplicated across Enemy, Spec_Enemy, and Boss.
- `Bullet` class serves dual purpose (enemy bullets + player skills).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tongthanhdat009)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tongthanhdat009)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
