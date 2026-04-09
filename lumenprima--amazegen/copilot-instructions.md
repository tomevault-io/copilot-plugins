## amazegen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-algorithm maze generator in Python. Generates, solves, visualizes, and exports mazes using 20+ algorithms. Algorithms are dynamically loaded from the `algorithms/` directory at runtime.

## Commands

```bash
# Generate mazes (default: 10x10, Prim's algorithm, PNG output)
python main.py

# Generate with specific algorithm and size
python main.py -x 20 -y 20 -a dfs -f png

# Generate with multiple algorithms and formats
python main.py -a dfs kru -f svg png -n 5 --print_stats

# Run all algorithms without output files (stats only)
python main.py -a all --no_output --print_stats

# Run test suite (uses SIGALRM — Linux only, won't work on Windows)
python test_algorithms.py

# Run single algorithm test
python test_kruskal_weighted.py

# Generate difficulty analysis plots
python plot_difficulty_vs_size.py
```

**Dependencies:** `cairosvg`, `Pillow` (no requirements.txt file exists — install manually with `pip install cairosvg Pillow`)

## Architecture

### Grid Representation

The maze uses a wall-based grid where a maze of logical size `width × height` maps to a grid of `(width*2+1) × (height*2+1)`. Grid cells are: `1` = wall, `0` = passage. Logical cell `(x, y)` maps to grid position `(y*2+1, x*2+1)`. Walls between adjacent cells are at the intermediate grid positions.

### Core Module Structure

- **`maze.py`** — `Maze` class: thin facade that delegates to specialized modules. Holds `grid`, `width`, `height`, `start`, `finish`, `algorithm`, `generate_func`, `cell_size`. The `generate()` method calls `generate_func(self)` then `select_start_finish(self)`.
- **`maze_algorithms.py`** — Legacy monolithic file containing duplicate implementations of many algorithms. The individual files in `algorithms/` are what actually get loaded and used at runtime.
- **`maze_utils.py`** — `select_start_finish(maze)` picks start/finish in diagonally opposite corners with weighted randomization. `calculate_difficulty(maze)` scores based on path length, dead ends, intersections, branching, and straight-line penalty.
- **`maze_pathfinding.py`** — A* search (`find_path`) using Manhattan distance heuristic. Returns list of `(row, col)` tuples in grid coordinates.
- **`maze_svg.py`** — SVG rendering: `to_svg()`, `overlay_solution()`, `to_svg_with_solution()`.
- **`maze_gcode.py`** — G-code output for CNC/pen plotters.
- **`utils.py`** — File I/O helpers: `svg_to_png()` (via cairosvg), `save_svg()`, `save_gcode()`.
- **`disjoint_set.py`** — Union-Find with path compression and union by rank, used by Kruskal-based algorithms.
- **`main.py`** — CLI entry point. Dynamically loads `algorithms/*.py`, builds `ALGO_MAP` with auto-generated short names (first letter of each word for multi-word names, first 3 chars for single-word).

### Algorithm Plugin System

To add a new algorithm, create `algorithms/<name>.py` with a function `generate_<name>(maze)`. It receives a `Maze` instance and must modify `maze.grid` in-place. The function is auto-discovered by `main.py` — no registration needed.

**Important:** `start`/`finish` coordinates use `(row, col)` ordering (y, x), not `(x, y)`. The pathfinding and SVG rendering depend on this convention. `Maze.generate()` only calls `select_start_finish` if `start`/`finish` are still `None` after the algorithm runs, so algorithms that set their own start/finish (like `recursive_division`) won't get them overwritten.

### Known Issues

- `test_algorithms.py` uses `signal.SIGALRM` which is not available on Windows.
- `maze_algorithms.py` contains legacy duplicate code that isn't used by the dynamic loader — the `algorithms/` directory files are the canonical implementations.
- No `requirements.txt` file exists despite being referenced in README.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LumenPrima)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LumenPrima)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
