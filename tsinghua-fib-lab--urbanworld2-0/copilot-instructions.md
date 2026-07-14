## urbanworld2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**urbanworld** — A pip-installable Python package for generating textured 3D urban scenes from OpenStreetMap data. Refactored from a collection of loose scripts into a clean package with shared modules, CLI interface, and proper dependency management.

## Development Commands

```bash
# Sync all dependencies (uses uv.lock for reproducibility)
uv sync --extra all

# Sync specific extras
uv sync --extra osm --extra vlm

# Run CLI
uv run urbanworld --help

# Run a Python script
uv run python -c "from urbanworld.common.geo import wgs84_to_bd09ll"

# Add a new dependency
uv add <package>

# Add to an optional group
uv add --optional mesh <package>

# Add dev dependency
uv add --group dev <package>

# Build for PyPI
uv build

# Publish to PyPI
uv publish

# Lint
uv run ruff check src/

# Test
uv run pytest
```

## Architecture

The package follows a stage-based pipeline architecture with shared common modules:

### `src/urbanworld/common/` — Shared Utilities (deduplicated)
- `geo.py` — Coordinate transforms (WGS84/GCJ-02/BD-09/BD-09MC), haversine distance
- `image.py` — Image encoding (base64), resizing, MIME detection
- `vlm.py` — `VLMClient` class wrapping OpenRouter API with retry logic
- `io.py` — Building metadata loading, structure annotations, confidence extraction
- `blender_helpers.py` — Blender import/export functions (conditional `bpy` import)
- `blender_runner.py` — Subprocess wrapper for invoking Blender scripts
- `parallel.py` — `iter_building_dirs()`, `run_parallel()` with tqdm

### Pipeline Stages
- `stage0_osm/` — OSM download (Blender wrappers) + height enrichment from GABLE
- `stage1_streetview/` — Baidu street view download + OWL-ViT building detection/cropping
- `stage2_vlm/` — VLM-based structure annotation, quality assessment, inpainting prep
- `stage3_imagination/` — VLM building facade imagination, reflection, regeneration
- `stage4_mesh/` — Hunyuan3D mesh generation + texture painting (optional)
- `stage5_assembly/` — Blender scene assembly, mesh replacement, rendering

### `blender_scripts/` — Blender-Only Scripts
These run inside Blender's Python interpreter via `blender -b -P <script> -- <args>`. They access shared modules through `URBANWORLD_PKG_PATH` environment variable injection (handled by `blender_runner.py`).

### `cli.py` — Click CLI
Entry point registered as `urbanworld` console script. Uses lazy imports — each subcommand imports its stage module only when invoked.

## Key Design Patterns

- **All stage modules expose a `run()` function** as the primary entry point
- **Blender scripts are bundled as package data** in `blender_scripts/`, invoked via subprocess wrappers in their respective stage modules
- **Configuration** is centralized in `config.py` (`UrbanWorldConfig` dataclass), loaded from env vars via `python-dotenv`
- **VLM calls** all go through `VLMClient` in `common/vlm.py` — never duplicate the HTTP/retry logic
- **Parallel mesh generation** uses `--partition`/`--total-partitions` args instead of separate p1-p8 script files

## Environment Variables

All configurable paths and API keys are in `.env.example`. Key ones:
- `OPENROUTER_API_KEY` — VLM inference
- `BAIDU_MAP_API_KEY` — Street view + coordinate conversion
- `BLENDER_PATH` — Blender executable
- `HUNYUAN3D_PATH` / `HUNYUAN3D_MODEL_PATH` — Mesh generation
- `OWLVIT_MODEL_PATH` — Building detection model

---
> Source: [tsinghua-fib-lab/UrbanWorld2.0](https://github.com/tsinghua-fib-lab/UrbanWorld2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
