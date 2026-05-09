## pltview

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pltview is a lightweight X11 viewer for AMReX plotfiles written primarily in C with a Python wrapper for installation. Designed for ultra-fast visualization of large scientific simulation datasets (primarily ERF — Energy Research and Forecasting) with minimal dependencies. Current version: see `pyproject.toml` `[project] version`.

## Development Commands

### Building
```bash
# Quick compile during development (produces ./pltview_c)
make

# Editable development install (recommended — produces ./pltview, wired to `pltview` CLI command)
pip install -e .
```

After `pip install -e .`, recompile with `make` or re-run `pip install -e .` to pick up C code changes.

### Testing
There is no test suite. Verify changes manually by running `pltview` against AMReX plotfile directories.

### Running
```bash
pltview plt00100                           # Single plotfile
pltview /path/to/output plt                # Multi-timestep mode
pltview --sdm plt00100                     # SDM particle mode
```

## Architecture

### File Layout
- **pltview.c**: Single-file C application (~7700 lines, no header files) — all visualization logic, X11/Athena widget GUI, AMReX plotfile parsing, and data rendering
- **pltview_entry.py**: Python entry point that locates and `execv()`s the compiled C binary
- **setup.py**: Custom setuptools build system (BuildC, InstallC, DevelopC, EditableWheel) that compiles the C binary during install
- **pyproject.toml**: Package metadata and version
- **pltview_pkg/**: Python package containing `__init__.py` with `get_binary_path()` helper; `pltview_bin` binary is placed here by `pip install`
- **map_layers/**: GeoJSON coastline/boundary overlays for map mode (auto-detected by geographic bounds)

### C Application Structure (pltview.c)
- **Data Structures**: `PlotfileData`, `Box`, `RGB`, `PlotData`, `PopupData`, `QuiverData` for managing AMReX data and visualization state
- **AMReX Parser**: Reads plotfile `Header`, `Level_*/Cell_H` box layouts, and `Cell_D_*` FAB binary data (double-precision, Fortran column-major order)
- **X11 GUI**: Athena widget-based interface (buttons, canvas, popup windows)
- **Rendering Engine**: Direct pixel manipulation for fast data visualization
- **Colormap System**: 8 built-in colormaps (viridis, jet, turbo, plasma, hot, cool, gray, magma)
- **Quiver System**: Vector field overlay with automatic component defaults per slice axis
- **Map Mode**: GeoJSON overlay rendering with lat/lon coordinate mapping
- **SDM Mode**: Super Droplet Method particle data histogram analysis

### Build System
- `make` produces `pltview_c`; `pip install -e .` produces `pltview` — both compiled from pltview.c
- Compiler flags: `-O3 -Wall -march=native`; linked against `-lX11 -lXt -lXaw -lXmu -lm`
- setup.py auto-detects X11 include/lib paths (`/usr/include/X11`, `/opt/X11/include`, `/usr/X11R6/include`)
- macOS requires XQuartz; include/lib paths at `/opt/X11/`

## Key Development Considerations

- **Single-file C architecture**: All code lives in pltview.c with no header files. Keep it that way.
- **No runtime dependencies** beyond X11 (no Python libraries needed at runtime)
- **Direct pixel manipulation** for performance — avoid adding abstraction layers that would slow rendering
- **Memory management**: Must handle large multi-timestep datasets; be careful with allocations especially in multi-timestep and SDM modes
- **AMR level handling**: Preserves slice positions when switching levels; higher levels overlay on coarser levels
- **Prerequisites**: X11 dev libraries (libX11-dev, libXt-dev, libXaw7-dev, libXmu-dev on Linux), XQuartz on macOS, gcc

---
> Source: [wang1202/pltview](https://github.com/wang1202/pltview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
