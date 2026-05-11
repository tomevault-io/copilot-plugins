## tenting

> This file provides guidance to AI when working with code in this repository.

# AGENTS.md

This file provides guidance to AI when working with code in this repository.

## Project Overview

Tenting is a Python tool for generating 3D-printable tented case rings for keyboard bottom plates. It extracts 2D outlines from 3D models (STL/STEP/OBJ/PLY/OFF) and generates ergonomic tenting rings with configurable tilt angles.

## Commands

```bash
# Run for specific keyboard
python generate_silakka_tenting.py
python generate_renga_tenting.py

# Run with custom parameters
python generate_renga_tenting.py --lateral-tilt 10 --wall-height 6

# See all CLI options
python generate_renga_tenting.py --help

# Skip saving and viewing (useful for testing)
python generate_renga_tenting.py --no-save --no-show

# Lint/format (Ruff, configured for on-save in VS Code)
ruff check .
ruff format .
```

## Architecture

**Two-layer design:**
- `helpers.py` - Generic library for any keyboard plate. Contains all core geometry functions.
- `generate_<keyboard>_tenting.py` - Keyboard-specific CLI applications (Silakka54, Renga54). Use as templates for other keyboards.

**Data flow:**
1. Load 3D model → slice near bottom to extract 2D outline
2. Simplify and center outline points (shapely)
3. Create plate solid via extrusion (CadQuery)
4. Generate ring with offset wires (inner clearance, outer wall, ledge)
5. Apply tilted cutting plane for ergonomic angles
6. Mirror for right side of split keyboard
7. Export to `results/` directory

**Key functions in helpers.py:**
- `create_plate_from_model()` - Entry point for geometry processing (handles multiple file formats)
- `tent_case_ring_from_plate_solid()` - Core ring generation algorithm. Supports `skip_upper_wall_indices` to cut openings (e.g., for TRRS sockets)
- `points_to_plate()` - Converts 2D outline to 3D extruded solid
- `mirror()` - Creates right-side variant across YZ plane
- `save_stl()` - Exports to results/ directory

**Tech stack:** CadQuery (OpenCASCADE wrapper), trimesh, shapely, scipy

## Development Notes

- Use OCP CAD Viewer VS Code extension for visual debugging
- Virtual environment at `venv/` with Python 3.13
- No automated tests - validation is visual via CAD viewer
- Output STLs go to `results/` directory
- Input models go to `data/` directory

---
> Source: [GreenKeebz/Tenting](https://github.com/GreenKeebz/Tenting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
