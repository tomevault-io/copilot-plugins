## luisa-cellcount-2025

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cell Counter is a CLI tool for analyzing cell segmentation data from microscopy images. It counts cells that are not within excluded regions and calculates cell density metrics.

## Commands

### Run the cell analysis
```bash
uv run python main.py --cells <cells_dir> --exclude <exclude_dir> [--pixel-size <float>]
```

Example:
```bash
uv run python main.py --cells data/CD8-CD137/Healthy\ Lung --exclude data/CD8-CD137/Healthy\ Lung\ Flaechen
```

### Debug/inspect segmentation files
```bash
uv run python debug_load.py <path_to_seg_npy_file>
```

## Architecture

- **main.py**: Core CLI application using Click
  - `analyze_cells()`: Main entry point that processes cell and exclusion directories
  - `save_results_to_excel()`: Outputs per-file results to timestamped Excel files

- **debug_load.py**: Utility for investigating `.npy` file structure

## Data Format

- Input: Cellpose `*_seg.npy` files containing dictionaries with a `masks` key
- Cell/exclusion directories must have matching file structures
- Exclusion files mark regions to ignore (mask value > 0)
- Output: Excel file with columns for parent folder, file path, total cells, area (µm²), and cell density

## Dependencies

Managed via uv with pyproject.toml: click, numpy, openpyxl

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrismuke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
