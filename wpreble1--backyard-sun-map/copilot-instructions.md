## backyard-sun-map

> Backyard DXF layout generator workflow


# Backyard DXF Workflow

`backyard_dxf/generate.py` defines the backyard layout in Python using ezdxf. Running `uv run python backyard_dxf/generate.py` produces `backyard_dxf/dupre_backyard.dxf`, which is opened in LibreCAD.

## Conventions

- All coordinates are in inches, origin at SW corner of the lot
- Movable objects (furniture, garden beds, etc.) are DXF BLOCKs so they can be grabbed and dragged as one piece in LibreCAD
- Block placements go in the BLOCK_INSERTS list at the top of generate.py
- Each layer has a distinct color; some layers use linetypes (e.g. dashed for clearances)
- After the user moves things in LibreCAD, read the DXF to find updated INSERT positions and sync generate.py

## Workflow

1. Read generate.py before making changes
2. Edit the Python data/code as requested
3. Regenerate the DXF by running `uv run python backyard_dxf/generate.py`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wpreble1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
