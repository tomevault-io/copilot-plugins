## eleego

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Eleego is a 3D printing experimentation toolkit for the Elegoo Neptune 4 Plus printer. It combines three main capabilities:

1. **Printer API Integration** - Connect to Fluidd/Moonraker API to query printer status and send commands
2. **Procedural Model Generation** - Generate parametric 3D models programmatically using trimesh
3. **G-code Analysis** - Document and understand slicer G-code sequences

## Environment Setup

This project uses `uv` for dependency management (not pip or poetry).

```bash
# Install dependencies
uv sync

# Run scripts
uv run python main.py
uv run python generate_model.py
```

Configuration is stored in `.env` (not tracked):
```env
API_KEY=your_moonraker_api_key
PRINTER_URL=http://192.168.1.141
```

## Printer Specifications (Neptune 4 Plus)

**Critical constraints for model generation:**
- Build volume: 331 × 331 × 387.1mm
- Nozzle: 0.4mm (all dimensions should be multiples of 0.4mm)
- Layer height: 0.2mm (heights should be multiples of 0.2mm)
- Wall thickness: Minimum 1.6mm (4 perimeters × 0.4mm nozzle)

## Architecture

### Printer API (`main.py`)

Connects to Moonraker API (Fluidd backend) using httpx async client:
- Default port: 80 (not 7125 - this printer runs on standard HTTP)
- Authentication: X-Api-Key header
- Key endpoints:
  - `/server/info` - Server status
  - `/printer/info` - Printer state
  - `/printer/objects/list` - Available objects
  - `/printer/objects/query` - Query specific objects

### Model Generation

**Two approaches available:**

1. **Simple examples** (`generate_model.py`) - Basic shapes without boolean operations
2. **Design projects** (`designs/XXX-name/`) - Parametric designs with versioning

**Boolean operations caveat:** trimesh.boolean requires Blender backend. If not available, use `trimesh.util.concatenate()` to export multiple meshes - the slicer will merge them.

### Design Project Structure

Each design follows this pattern:
```
designs/XXX-name/
├── prompt.md              # Design requirements and specifications
├── generate.py            # Main generator (or generate_vX.py for versions)
├── generate_previews.py   # Generate PNG previews from STL
├── model-name.stl         # Output STL files
├── *_preview.png          # Preview images for documentation
└── README.md              # Design documentation with images
```

**Design workflow:**
1. Create `prompt.md` with specs (dimensions, nozzle constraints, features)
2. Write parametric generator script with configurable parameters
3. Generate multiple versions by adjusting parameters
4. Create preview images: `uv run python generate_previews.py`
5. Document in README with comparison tables

### G-code Templates

Located in `gcode_templates/` with inline comments explaining each command:
- `start_gcode.gcode` - Two-phase heating, homing, purge line
- `end_gcode.gcode` - Retraction, parking, shutdown
- `other_codes.gcode` - Layer change, M600 filament change

**Slicer variables** (replaced at slice time):
- `[bed_temperature_initial_layer_single]`
- `[nozzle_temperature_initial_layer]`
- `[print_bed_max[0]]` - X dimension (331mm)
- `{max_layer_z}`, `{printable_height}`, etc.

## Key Commands

```bash
# Check printer status
uv run python main.py

# Generate example models
uv run python generate_model.py

# Generate specific design
cd designs/001-glue-stick-holder
uv run python generate_v4_hexagon.py

# Generate preview images
cd designs/001-glue-stick-holder
uv run python generate_previews.py
```

## Design Principles

**For new parametric designs:**

1. All dimensions as multiples of 0.4mm (nozzle width)
2. Heights as multiples of 0.2mm (layer height)
3. Minimum wall thickness: 1.6mm (4 perimeters)
4. Design to avoid supports (overhangs ≤ 45°, upward-pointing features)
5. Include parameter comments explaining the math (e.g., "4 perimeters × 0.4mm")
6. Create multiple versions by varying key parameters
7. Generate preview images for documentation
8. Document print settings: estimated time, material, supports needed

**Preview image generation:**
Uses matplotlib for 3D visualization. Standard settings:
- Resolution: 800×600
- View angle: elev=25, azim=45
- Colors: facecolor='lightblue', edgecolor='navy'

## Common Patterns

**Creating hollow cylinders:**
```python
outer = trimesh.creation.cylinder(radius=outer_r, height=h, sections=64)
inner = trimesh.creation.cylinder(radius=inner_r, height=h-base_thickness, sections=64)
inner.apply_translation([0, 0, base_thickness])
holder = trimesh.boolean.difference([outer, inner], engine='blender')
```

**Hexagonal shapes:**
```python
hexagon = trimesh.creation.cylinder(radius=r, height=h, sections=6)  # 6 sides
```

**Exporting STL:**
```python
mesh.export("output.stl")
```

## Notes

- Moonraker API runs on port 80 for this printer (not the typical 7125)
- Boolean operations fail gracefully - use concatenation fallback
- STL file sizes vary: simple shapes ~5KB, complex ~40KB, previews ~150KB
- Generated models go to `generated_models/` (tracked in git)
- Design-specific models stay in their design folders (tracked in git)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nim444) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
