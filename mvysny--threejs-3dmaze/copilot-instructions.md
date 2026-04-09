## threejs-3dmaze

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Dungeon Maze — a first-person 3D dungeon crawler (Wolfenstein-style) built with Three.js. Zero-build, pure JavaScript project with no package manager or bundler. Three.js v0.170.0 is loaded via CDN importmap in index.html.

## Commands

**Run locally** (any static file server):
```sh
npx serve .
```

**Run tests:**
```sh
node --test mazegen.test.js
```

Tests use Node.js built-in test runner (node:test) — no external dependencies.

## Architecture

Two files contain all the code:

- **`mazegen.js`** — Pure maze generation module (no DOM/Three.js dependency). Exports `Maze` class and `generateMaze(mode)` function. Two generation modes:
  - `'classic'` — 21×21 recursive backtracker maze
  - `'rooms'` — 41×41 rooms-and-corridors algorithm (room placement → MST connectivity → BFS corridor carving → door/column placement)
- **`index.html`** — Everything else: Three.js scene setup, procedural texture generation (canvas 2D), player movement/collision, door mechanics, minimap, and game loop. Imports from mazegen.js.

### Maze grid

2D array of cell types: `CELL_OPEN=0`, `CELL_WALL=1`, `CELL_START=2`, `CELL_EXIT=3`, `CELL_DOOR=4`. Walls carry texture indices via `Maze.wallTextureMap`. Doors track position and orientation in `Maze.doors` array.

### Rendering

Walls are grouped by texture and merged into single `BufferGeometry` per texture for performance. Doors are animated meshes (portcullis-style, auto-close after 3s). Game loop is capped at 60 FPS.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mvysny)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mvysny)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
