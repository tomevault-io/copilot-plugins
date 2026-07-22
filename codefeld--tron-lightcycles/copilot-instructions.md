## tron-lightcycles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TRON Lightcycles is a pygame-based implementation of the classic TRON light cycle game. Players control motorcycles that leave light trails behind them, and the objective is to avoid crashing into walls, trails, or obstacles while making opponents crash.

## Running the Game

```bash
# Run the game
python3 main.py
```

The game uses a virtual environment (`.venv`) for dependencies. Dependencies are managed via `uv` (see `pyproject.toml` and `uv.lock`).

To install/update dependencies:
```bash
uv sync
```

## Game Architecture

### Core Game Loop Structure

The game follows a typical pygame event loop pattern in `main.py`:

1. **Main Menu** (`main_menu()`) - Shows initial screen where players choose 1-player (vs AI) or 2-player mode
2. **Game Reset** (`reset_game()`) - Initializes a new round with countdown, spawns obstacles, clears powerups
3. **Main Loop** - Runs at 60 FPS, handles input, movement, collision detection, and rendering
4. **Game Over State** - Shows winner, updates scores, allows continuing or returning to menu

### Player Movement System

The movement system uses fractional pixel stepping to ensure smooth collision detection at varying speeds:

- **Position Tracking**: Each bike has a back position (`p1_pos`, `p2_pos`) and direction vector (`p1_dir`, `p2_dir`)
- **Front Position Calculation**: `get_front_pos()` computes the front of the bike based on sprite width and direction
- **Stepped Movement**: `step_move_player()` moves bikes pixel-by-pixel, checking collisions at the front position each step
- **Trail System**: Each bike leaves a trail of grid positions in `p1_trail` and `p2_trail` lists

### Starting Position Logic

`reset_sprites()` contains complex logic to prevent bikes from spawning in adjacent corners (main.py:92-168). When one bike spawns in a corner, the other bike cannot spawn in the adjacent corner to avoid immediate collisions.

### Collision Detection

Multiple collision types are checked every frame:

1. **Wall collisions** - Front position out of bounds
2. **Trail collisions** - Front position intersects any trail segment
3. **Obstacle collisions** - Front position intersects randomly spawned obstacles
4. **Bike-to-bike collisions** - Head-on (fronts touch) or side impacts (front hits body)

The `check_collision()` function handles wall and trail checks. Bike-to-bike collision uses distance calculations between front positions and tight hitboxes (main.py:906-945).

### Power-Up System

Power-ups spawn every ~5 seconds (`POWERUP_SPAWN_INTERVAL`):

- **Freeze** (cyan) - Freezes opponent for 3 seconds (cannot turn or move)
- **Slow** (green) - Slows opponent to half speed for 5 seconds

Player status is tracked in `p1_status` and `p2_status` dictionaries with `frozen_until` and `slow_until` timestamps.

### AI Player

Single-player mode uses `ai_control()` (main.py:613-658) which implements a simple lookahead algorithm:

- Checks 12 steps ahead in current direction
- If collision predicted, picks a random safe direction
- Avoids 180-degree turns
- Turn cooldown prevents excessive turning

There's a commented-out enhanced AI implementation (main.py:660-771) that attempts to seek power-ups using BFS pathfinding.

### Rendering System

Custom rendering for TRON aesthetic:

- **Grid Background**: `draw_tron_grid()` renders a glowing grid on dark background
- **Sprite Rotation**: `blit_bike_with_front_at()` handles rotating bike sprites based on direction while keeping the back position fixed
- **Trail Rendering**: Trails drawn as colored rectangles at block size (5px)
- **Obstacles**: Black rectangles with white outlines

### Music System

The game has dynamic music tied to game states:

- **Menu**: armory.mp3
- **Countdown**: clu.mp3
- **Gameplay**: Random selection from [derezzed.mp3, fall.mp3, disc_wars.mp3, the_game_has_changed.mp3]
- **Round Win**: the_grid.mp3 (or rinzler.mp3 for AI wins in single-player)
- **Match Win**: end_titles.mp3 (or adagio_for_tron.mp3 for AI match win)
- **Draw**: arena.mp3
- **Collision**: derezzed_sound.mp3 (sound effect)

All music files are optional - the game checks if files exist before loading them using `Path.exists()`.

## Key Constants and Configuration

- `WIDTH, HEIGHT = 900, 900` - Game window dimensions
- `SPEED = 5` - Base movement speed in pixels per frame
- `BLOCK_SIZE = 5` - Trail segment size
- `MAX_SCORE = 5` - First to 5 wins the match
- `turn_cooldown = 50` - Minimum milliseconds between turns
- `POWERUP_SPAWN_INTERVAL = 5000` - Power-up spawn rate

## Controls

- **Player 1 (Blue)**: WASD keys
- **Player 2 (Orange)**: Arrow keys (or AI in single-player mode)
- **Menu**: Press 1 or 2 to select game mode
- **Game Over**: SPACE to continue, ESC to return to menu

## Assets

The game requires these asset files (all checked with `Path.exists()` for graceful degradation):

- `blue_cycle.png`, `orange_cycle.png` - Bike sprites
- `grid.jpg` - Background image
- Multiple `.mp3` files for music and sound effects

## Code Organization

The entire game is implemented in a single `main.py` file (~1000 lines):

- Lines 1-48: Imports, initialization, global state
- Lines 50-91: Sprite rendering utilities
- Lines 92-174: Player spawn logic and status reset
- Lines 176-213: Main menu
- Lines 215-258: Grid rendering
- Lines 260-309: Game setup and configuration
- Lines 311-382: UI and scoreboard
- Lines 383-437: Game reset and obstacle generation
- Lines 439-494: Power-up system
- Lines 496-525: Countdown sequence
- Lines 527-611: Win condition handlers
- Lines 613-771: AI control (with commented enhanced version)
- Lines 773-815: Stepped movement system
- Lines 817-1004: Main game loop

## Development Notes

- The project uses Python 3.13+ and pygame 2.6.1+
- Dependency management via `uv` package manager
- No formal test suite exists
- Game state is managed through global variables
- All game logic runs synchronously in a single event loop

---
> Source: [codefeld/tron_lightcycles](https://github.com/codefeld/tron_lightcycles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
