## brainfit-paper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A research paper codebase analyzing associations between memory performance, physical fitness (Fitbit data), and mental health in ~113 participants. The pipeline runs end-to-end from raw CSV files → preprocessed pickles → figures for the paper.

## Running the notebooks

**With Docker (recommended for full reproducibility):**
```bash
docker build --platform linux/amd64 -t brainfit .
docker run --rm -it --platform linux/amd64 -p 8888:8888 -v $PWD:/mnt brainfit
```
Copy the `http://127.0.0.1:8888/?token=...` URL into a browser.

**Without Docker (works for most notebooks):**
```bash
pip install pandas numpy seaborn matplotlib scipy scikit-learn tqdm jupyter
cd code
jupyter notebook
```
`behavioral_data.ipynb` additionally requires `quail`, `hypertools`, and `datawrangler` (ContextLab packages). On macOS, `brainiak` also requires `brew install mpich`.

Note: the Dockerfile uses `debian:stretch` (EOL). The `apt-get` steps require redirecting to `archive.debian.org` and passing `-o Acquire::Check-Valid-Until=false` — this fix is already applied to the local Dockerfile.

## Architecture

### Data flow

```
data/raw_formatted/BFM_AMT_*.csv   (113 participant files)
        ↓  loader.load_raw() → loader.get_formatted_data()
        ↓  parse_data() splits columns into: fitbit / survey / experiment / meta
        ├── behavioral_stats()  → behavioral_summary.pkl
        ├── fitness_stats()     → fitness_summary.pkl
        └── survey_stats()      → inline DataFrames in notebooks
                ↓
        Notebooks load pickles, generate figures, save to data/preprocessed/
```

### Modules in `code/`

- **loader.py** — the backbone. All data loading, parsing, and feature extraction lives here. Expensive operations (UMAP embeddings, event segmentation) are cached as `.pkl` files under `data/preprocessed/embeddings/`.
- **fitness.py** — single utility: `format_text()` for axis label formatting.
- **free_recall.py** — plotting functions for the free-recall memory task (SPC, lag-CRP, fingerprints, PFR).
- **naturalistic_recall.py** — UMAP trajectory embedding and consistency testing for the movie-watching recall task. Contains a small computational geometry library (Point, LineSegment, Circle, Rectangle) used for trajectory intersection counting during UMAP hyperparameter search.

### Notebooks (run in this order)

1. `demographics.ipynb` — Figure S1, no heavy deps
2. `behavioral_data.ipynb` — Figures 2–5, S2–S5; saves `behavioral_summary.pkl`
3. `fitness_data.ipynb` — Figures 4, S6–S10; saves `fitness_summary.pkl`
4. `exploratory_analysis_correlations.ipynb` — Figures 5, S11–S14; bootstrap correlations (10k samples)
5. `reverse_correlation_analysis.ipynb` — Figures 6, S15–S20; time-varying weighted averages

### Key conventions

- **Caching pattern**: functions in `loader.py` check for a `.pkl` file before computing; delete the pickle to force recomputation.
- **`get_formatted_data()`** is the main entry point used by all notebooks — it calls `behavioral_stats()`, `fitness_stats()`, and `survey_stats()` and returns a merged DataFrame.
- **`datawrangler` (`import datawrangler as dw`)** is the ContextLab package `pydata-wrangler` on PyPI. It handles text embeddings (BERT, LDA, GloVe) used in naturalistic recall analysis.
- The project is referred to interchangeably as BrainFit, FitBrain, and FitWit throughout the code — these all mean the same study.

---
> Source: [kate-marine/brainfit-paper](https://github.com/kate-marine/brainfit-paper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
