## bay-window-maker

> This file provides project-specific guidance for Claude Code when working in this repository.

# CLAUDE.md

This file provides project-specific guidance for Claude Code when working in this repository.

## Project Overview

Bay Window Maker is a Python CLI tool that:
1. Calculates valid 3-part bay window layouts (centre face + two symmetric side faces) from stock double-hung window units
2. Exports results as JSON
3. Renders detailed SVG construction sheets (plan view + elevations + dimension callouts) from that JSON

All measurements are in **inches**.

## Repository Layout

```
bay_window_calculator_with_svg.py   # Calculator: search / verify commands, JSON output
bay_window_svg_renderer.py          # Renderer: reads JSON, writes per-candidate SVG sheets
bay_window_calculator.py            # Older standalone calculator (no SVG, kept for reference)
test_bay_window_calculator.py       # pytest suite (111 tests) for the calculator module
pyproject.toml                      # uv project config
uv.lock                             # Pinned lockfile — always commit this
instructions.txt                    # Original design notes
output_svgs/                        # Generated artefacts — gitignored
```

## Environment & Commands

This project uses `uv` exclusively. Never use `pip install` or activate a venv manually.

| Task | Command |
|------|---------|
| Install / sync deps | `uv sync` |
| Run tests | `uv run pytest` |
| Run calculator | `uv run python bay_window_calculator_with_svg.py <search\|verify> ...` |
| Run renderer | `uv run python bay_window_svg_renderer.py --input-json <file> --output-dir <dir>` |

### Typical two-step workflow

```sh
uv run python bay_window_calculator_with_svg.py search \
  --opening-width 72 --opening-height 36 \
  --projection-depth 16 --side-angle-deg 30 \
  --limit 5 \
  --json-output output_svgs/results.json \
  --svg-dir output_svgs

uv run python bay_window_svg_renderer.py \
  --input-json output_svgs/results.json \
  --output-dir output_svgs
```

## Key Design Decisions

- The **side faces** create the bay projection; the centre face is square to the wall.
- All windows in a candidate share the same height (`opening_height`).
- Left and right side windows are always identical.
- Centre face supports 1–4 equal-width units separated by mullion posts.
- `BayConstraints` is the single source of truth for all framing parameters; it is embedded in JSON output under the `"constraints"` key.
- `sill_height` and `frame_body_depth` are optional constraint fields that feed through to JSON and SVG rendering.

## Module Responsibilities

### `bay_window_calculator_with_svg.py`

Core dataclasses: `WindowUnit`, `BayConstraints`, `FaceDimensions`, `BayCandidate`, `BaySvgLayout`, `FacePolygon`

Key functions:
- `calculate_single_window_face` / `calculate_center_face` — framing maths
- `find_candidates` — exhaustive search, returns scored/sorted list
- `verify_candidate` — validate and calculate one specific layout
- `candidate_to_dict` — serialise to JSON-compatible dict; pass `constraints=` to embed the constraint block
- `write_candidates_json` / `write_candidate_json` — write JSON files
- `render_candidate_svg` / `write_svg_file` — simple plan-view SVG
- `build_default_stock_windows` / `load_stock_windows_from_json` — stock window sources

CLI subcommands: `search`, `verify`. Shared flags via an argparse parent parser.

### `bay_window_svg_renderer.py`

Reads the JSON produced by the calculator and renders a detailed multi-panel SVG sheet per candidate (plan view, front elevation, side elevation, spec table).

Key dataclasses: `BayRenderSpec`, `FaceSpec`, `WindowSpec`
Entry point: `load_specs(path)` → list of `BayRenderSpec`

The renderer is **read-only with respect to the calculator module** — it never imports from it.

## Testing

Tests live in `test_bay_window_calculator.py` and cover the calculator module only. The renderer has no automated tests yet.

Run and check:
```sh
uv run pytest -v
```

All 111 tests must pass before committing. When adding new public functions to the calculator, add corresponding tests.

## Git Conventions

- Commit `uv.lock` — this is an application, reproducible installs matter.
- The `output_svgs/` directory is gitignored; never commit generated SVG or JSON files.
- Branch: `main` is the default and only branch currently.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/conradstorz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/conradstorz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
