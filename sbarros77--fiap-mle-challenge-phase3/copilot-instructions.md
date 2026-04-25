## fiap-mle-challenge-phase3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

FIAP MLE Phase 3 Challenge — flight delay analysis and prediction on a US flights dataset. The goal is to build a complete data science pipeline covering supervised and unsupervised ML, from EDA through model interpretation.

**Student**: Sandro Barros | **RM**: RM-361869

## Commands

```bash
# Install dependencies
uv sync

# Run the main script
uv run python main.py

# Run tests
uv run pytest

# Run a single test file
uv run pytest tests/test_foo.py

# Lint
uv run ruff check .

# Format
uv run ruff format .

# Add a runtime dependency
uv add <package>

# Add a dev dependency
uv add --dev <package>

# Run individual pipeline modules directly
uv run python -m pipelines.download_data.download_kaggle
uv run python -m pipelines.download_data.download_gdrive
uv run python -m pipelines.profiler.profile
uv run python -m pipelines.classification_data_preparation.prepare
uv run python -m pipelines.clustering_data_preparation.prepare
uv run python -m pipelines.clustering_data_preparation.airport_features

# Supervised — trains XGBoost + Logistic Regression, prints comparison, saves models
uv run python -m pipelines.model_training.supervised.classification_trainer
# Or train individually:
uv run python -m pipelines.model_training.supervised.xgb_trainer
uv run python -m pipelines.model_training.supervised.lr_trainer

# Unsupervised — K-Means with auto K search (elbow + silhouette), saves model + plot
uv run python -m pipelines.model_training.unsupervised.kmeans_trainer

# Inference — build lookup table (run once after training, or after retraining)
uv run python -m pipelines.inference.build_lookup
# Inference — predict a single flight via CLI
uv run python -m pipelines.inference.predict \
  --airline AA --origin LAX --destination JFK \
  --month 7 --day-of-week 2 \
  --scheduled-departure 800 --scheduled-time 330 --distance 2475
```

## Stack

- **Python**: 3.13 (pinned in `.python-version`)
- **Package manager**: `uv` (lockfile: `uv.lock`, config: `pyproject.toml`)
- **Data processing**: Polars + PyArrow
- **ML**: scikit-learn, xgboost
- **Profiling**: ydata-profiling
- **Data download**: kaggle (API), gdown (Google Drive)
- **Database**: psycopg2-binary (PostgreSQL)
- **Testing**: pytest
- **Linter/Formatter**: ruff

## Data

Raw data lives under `data/source/` (not tracked in git):
- `flights.csv` (~592 MB) — main dataset with US flight records
- `airlines.csv` — airline lookup table
- `airports.csv` — airport lookup table

Download via Kaggle (`sriharshaeedala/airline-delay`) or Google Drive folder `1aS7exW5N0qq1uIxvIBcAfc18OHojOMjj`. Kaggle requires `~/.kaggle/kaggle.json` or `KAGGLE_USERNAME`/`KAGGLE_KEY` env vars.

## Architecture

Each pipeline module is independently runnable (has `if __name__ == "__main__"` blocks) and uses path anchoring via `Path(__file__).parents[N]` to resolve `data/` relative to the repo root — not relative to cwd.

- `main.py` — entry point (currently a placeholder)
- `pipelines/download_data/` — two download strategies: `download_kaggle.py` and `download_gdrive.py`
- `pipelines/classification_data_preparation/prepare.py` — engineers classification features from `flights.csv`; outputs `data/training/classification_flights.csv`
- `pipelines/clustering_data_preparation/prepare.py` — engineers clustering features (post-flight actuals + aggregations) from `flights.csv`; outputs `data/training/clustering_flights.csv`
- `pipelines/profiler/profile.py` — unified analysis and profiling pipeline; runs ydata-profiling on all CSVs (files >100 MB sampled to 100k rows in minimal mode) and computes delay cause distribution for `flights.csv`; outputs are split by type: `data/profile/source_data/` (HTML per source CSV, `delay_causes.json`, `delay_causes.png`) and `data/profile/training_data/` (HTML per training CSV)
- `pipelines/model_training/supervised/` — binary classifiers for `IS_DELAYED`:
  - `base.py` — shared helpers: `load_data`, `build_preprocessor` (OrdinalEncoder + SimpleImputer + StandardScaler via ColumnTransformer), `evaluate`, `save_artifacts`; `ROOT = Path(__file__).parents[3]`
  - `xgb_trainer.py` — XGBoost classifier with `scale_pos_weight` for class imbalance; saves `models/xgb_classifier.ubj` + `xgb_preprocessor.joblib`
  - `lr_trainer.py` — Logistic Regression with `class_weight="balanced"`; saves `models/lr_classifier.joblib` + `lr_preprocessor.joblib`
  - `classification_trainer.py` — orchestrator: trains both, prints side-by-side Accuracy / F1 / ROC-AUC comparison
- `pipelines/model_training/unsupervised/` — K-Means clustering for `clustering_flights.csv`:
  - `base.py` — shared helpers: `load_data` (returns features + identity columns separately), `build_preprocessor` (StandardScaler), `save_artifacts`; `ROOT = Path(__file__).parents[3]`
  - `kmeans_trainer.py` — searches K=2–10 via elbow + silhouette (sample 50k), picks best K, trains final model, prints per-cluster summary, saves `models/kmeans_model.joblib` + `kmeans_preprocessor.joblib` + `data/profile/kmeans_elbow_silhouette.png`
- `pipelines/inference/` — flight delay inference using the trained XGBoost model:
  - `build_lookup.py` — one-time job: aggregates `IS_DELAYED` mean by `(ORIGIN_AIRPORT, MONTH)` from `classification_flights.csv`; outputs `data/inference/origin_month_delay_rates.json` (3,722 combinations + global mean fallback); must be re-run after retraining
  - `loader.py` — loads model (via `.load_model()`), preprocessor (`joblib`), and lookup JSON from disk
  - `feature_engineering.py` — replicates `_add_season`, `_add_dep_cyclic`, and `_add_origin_date_delay_rate` from `classification_data_preparation/prepare.py`; assembles the exact feature matrix expected by the preprocessor
  - `predict.py` — `predict_single()` / `predict_batch()` Python API + argparse CLI; supports `--threshold` flag (default 0.5) for binary label cutoff; returns `PredictionResult(delay_probability, is_delayed, label)`
- `data/inference/` — inference artifacts: `origin_month_delay_rates.json`
- `models/` — trained model artifacts (not tracked in git: `*.ubj`, `*.joblib`)
- `pipelines/analytics/airports.ipynb` — exploratory notebook for airport-level analysis

Tests live in `tests/`; pytest is configured via `[tool.pytest.ini_options]` in `pyproject.toml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sbarros77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
