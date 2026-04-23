## asee-2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ASEE 2026 conference paper analyzing interdisciplinary overlap between STEM domains using the MIDFIELD student enrollment dataset. The project computes seven similarity metrics (Jaccard, TF-IDF Cosine, Overlap Coefficient, PMI, Max-Scaled Cosine, Dice, Kulczynski) across four analysis modes: computing, engineering, sciences, and interdisciplinary.

## Build/Render

```bash
quarto render analysis/index.qmd   # Render the main analysis document to IEEE PDF
```

Output: IEEE-formatted PDF, heatmap PNGs, and similarity metrics JSON files. The `_freeze/` directory caches R code execution results.

## Interactive Explorer

Live at https://ransomts.github.io/ASEE-2026/ — deployed via GitHub Pages from `site/`. The `site/qrcode.png` links to this URL for the conference poster.

## Tech Stack

- **Quarto** with the `dfolio/ieee` extension for IEEE journal formatting (in `_extensions/`)
- **R 4.5** managed via `renv` (lockfile: `renv.lock`, activated by `.Rprofile`)
- **Core R packages**: `data.table` (primary data manipulation), `lsa` (cosine similarity), `pheatmap` (heatmaps), `jsonlite` (export), `tidyverse`/`stringr` (string processing)
- **LaTeX** via `pdflatex` for PDF generation

## Directory Structure

```
paper/              — WIP paper prose, bibliography, reviews, presentation
  asee_wip.org      — Org-mode paper source
  asee_wip.pdf      — Compiled PDF
  bibliography.bib  — Citations
analysis/           — R pipeline and outputs
  index.qmd         — Main analysis (data loading, metrics, heatmaps, JSON export)
  make_heatmaps.R   — Standalone heatmap script
  heatmaps/         — Generated heatmap PNGs
  metrics/          — Similarity metrics JSON files
site/               — Interactive explorer (GitHub Pages)
  index.html        — Single-page app
  data/             — Pre-computed JSON with metrics, course counts, dendrograms
  qrcode.png        — QR code for conference poster
data/               — MIDFIELD source data (gitignored, restricted)
```

## Architecture

The analysis pipeline in `index.qmd` follows this flow:

1. **Data loading**: Reads MIDFIELD degree and course CSVs via `data.table::fread`, normalizes course names (uppercase, strip leading zeros, remove suffix letters like H/W)
2. **Vectorization**: Merges degree/course tables, computes per-major course frequencies and probabilities (`prob_taken`, `max_scaled`)
3. **Analysis mode selection**: Set `analysis_mode` to one of `"computing"`, `"engineering"`, `"sciences"`, `"interdisciplinary"` — each filters to different CIP code sets
4. **Metric computation**: Calculates all seven similarity matrices for the selected CIP codes
5. **Visualization**: Generates `pheatmap` heatmaps saved as `heatmap_{mode}_top{N}_{metric}.png`
6. **Export**: Writes all matrices to `similarity_metrics_{mode}_top{N}.json`

The code block starting at the `# consolidated` section is the self-contained analysis that handles all modes. The `analysis_mode` and `n_top_limit` variables at the top of that block control what gets generated. To produce all outputs, render once per mode.

## CIP Code Domains

- **Computing**: CIP 11xxxx + 1409xx (Computer Engineering) + 1512xx/1513xx (Engineering Tech)
- **Engineering**: CIP 14xxxx + 15xxxx, sorted by enrollment size
- **Sciences**: CIP 26 (Bio), 27 (Math), 40 (Physical Sciences), 30 (Interdisciplinary), 01 (Ag), 03 (Natural Resources)
- **Interdisciplinary**: Hand-picked representatives (CS, MechEng, Bio, Math, Physics, Geology)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ransomts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
