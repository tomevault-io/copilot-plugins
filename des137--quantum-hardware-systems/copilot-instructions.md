## quantum-hardware-systems

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository characterizes and analyzes quantum computing hardware across global vendors. It contains two main components:

1. **`src/`** — Core data models (`models.py`) defining dataclasses for quantum systems (platform types, fidelities, coherence times, DiVincenzo scores, Quantum Readiness Index, advantage gap analysis). `quantum_database.py` is currently empty.

2. **`ibm_miami/`** — A complete analysis pipeline for the IBM Miami 120-qubit superconducting processor, built around calibration CSV data. The modules form a layered architecture:
   - `ibm_miami_loader.py` — CSV parsing, CZ error/gate length string parsing (format: `"neighbor:value;neighbor:value;..."`), edge extraction
   - `ibm_miami_stats.py` — Statistical analysis (IQR outlier detection, correlation, T1/T2 ratio checks, `StatsSummary` dataclass)
   - `ibm_miami_viz.py` — Matplotlib visualizations (histograms, heatmaps, connectivity graphs)
   - `ibm_miami_report.py` — PDF report generation via `PdfPages`, includes physics explanations of each calibration parameter
   - `ibm_miami_analysis.py` — Top-level `MiamiAnalyzer` class that ties everything together

3. **`specs/`** — Reference documentation with detailed hardware specifications for 20+ quantum systems (error rates, coherence times, quantum volume) sourced from ArXiv papers and company docs.

## Key Conventions

- The `ibm_miami/` modules use **relative imports between siblings** (e.g., `from ibm_miami_loader import ...`). Scripts must be run from within the `ibm_miami/` directory.
- CZ errors and gate lengths are stored as semicolon-separated `neighbor:value` pairs in CSV columns and parsed into dicts by `parse_cz_errors()` and `parse_gate_lengths()`.
- The `MiamiAnalyzer` class caches computed values (`_cz_errors`, `_gate_lengths`, `_stats_cache`).
- Composite qubit quality scores use z-score normalization across error metrics plus inverse coherence times.

## Running the Analysis

```bash
# All scripts assume CWD is ibm_miami/
cd ibm_miami

# Run full CLI analysis (finds CSV automatically)
python ibm_miami_analysis.py

# Generate PDF report
python ibm_miami_report.py

# Run stats summary
python ibm_miami_stats.py
```

### Jupyter/interactive usage
```python
from ibm_miami_analysis import MiamiAnalyzer
analyzer = MiamiAnalyzer('ibm_miami_calibrations.csv')
analyzer.summary()
analyzer.plot_all()
```

## Data Pipeline & README Generation

`README.md` is **auto-generated** from `data/hardware_data.json`. Do not edit README.md directly.

```bash
# Edit the structured data file, then regenerate:
#   1. Edit data/hardware_data.json
#   2. Run:
python scripts/update_hardware_data.py --readme-only

# Full update (fetches company pages, Wikipedia, news, arXiv, then regenerates):
python scripts/update_hardware_data.py

# Dry run (shows what would change):
python scripts/update_hardware_data.py --dry-run
```

A weekly GitHub Action (`.github/workflows/update-quantum-hardware.yml`) runs the updater every Monday and opens a PR if changes are detected. The updater uses heuristic change detection — all detected changes require human review before merging.

## Dependencies

Python packages: `pandas`, `numpy`, `scipy`, `matplotlib`. No requirements.txt or setup.py exists — install manually. The update scripts (`scripts/`) use only the Python standard library.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/des137) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
