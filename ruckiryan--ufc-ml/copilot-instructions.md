## ufc-ml

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The project is a UFC fight outcome prediction system built in Python. It includes:
- an XGBOOST binary classification model to predict fight winners.
- A web scraper to collect historical fight data from ufcstats.com.
- Data cleaning and feature engineering scripts to prepare the dataset.
- Prediction scripts for upcoming events that calculate win probabilities and expected value against betting lines.

## Setup

```bash
# Python 3.12.x required (.python-version pins 3.12.12)
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt        # core: scraper, training, prediction
pip install -r requirements-dev.txt    # adds openpyxl, JupyterLab, matplotlib, seaborn, scipy
```

Dependencies are managed with **pip-tools**. Edit `requirements.in` or `requirements-dev.in`, then recompile and sync:

```bash
pip-compile --strip-extras --no-emit-trusted-host --no-header requirements.in -o requirements.txt
pip-compile --strip-extras --no-emit-trusted-host --no-header requirements-dev.in -o requirements-dev.txt
pip-sync requirements-dev.txt
```

## Common Commands

```bash
# Scrape fight data from ufcstats.com
python -m src.scraper                  # All events → data/raw_fights.csv
python -m src.scraper --max-events 5   # Quick test run

# Data pipeline
python src/clean_ufc_data.py           # data/large_dataset.xlsx → data/ufc_features.csv

# Train models
python src/train_lite_modelV2.py       # Recommended: 15-feature lite model → models/ufc_xgb_lite.joblib
python src/train_model.py              # Full model → models/ufc_xgb_model.joblib

# Predict upcoming event (copy/adapt an existing predict_*.py script)
python src/predict_ufc325.py           # → data/ufc325_predictions.csv

# Model analysis
python src/feature_importance.py       # → visuals/feature_importance.csv
python src/show_features.py            # List features in trained model
```

There are no automated tests or linting configured.

## Docker

A multi-stage `Dockerfile` produces two images (`runtime` and `dev`). `docker-compose.yml` defines three services:

| Service | Stage | Invocation |
|---|---|---|
| `scraper` | runtime | `docker compose run --rm scraper` |
| `ml` | runtime | `docker compose run --rm ml python src/<script>.py` |
| `jupyter` | dev | `docker compose up jupyter` → http://localhost:8888 |

```bash
docker compose build                                               # build all images
docker compose run --rm scraper                                    # run scraper
docker compose run --rm ml python src/train_lite_modelV2.py       # train model
docker compose run --rm ml python src/predict_ufc325.py           # predict event
docker compose up jupyter                                          # start notebook server
docker compose down                                                # stop all
```

`data/`, `models/`, `notebooks/`, and `visuals/` are bind-mounted — outputs persist on the host.

## Architecture

**Data flow**: ufcstats.com → `src/scraper.py` → `data/raw_fights.csv` → `src/clean_ufc_data.py` → `data/ufc_features.csv` → `src/train_lite_modelV2.py` → `models/ufc_xgb_lite.joblib` → `src/predict_*.py`

**Two model variants**:
- **Full model** (`train_model.py`): Uses all available features from cleaned data
- **Lite model V2** (`train_lite_modelV2.py`): 15 curated features — age, height, reach, striking, takedown, submission, and win/loss differentials. This is the current production model used by all prediction scripts.

**Feature engineering** (`clean_ufc_data.py`): Creates `*_diff` columns (red minus blue) for each stat, one-hot encodes weight class, and sets binary target `win_red`.

**Prediction scripts** (`predict_*.py`): Each event gets its own script. Input is a manually-prepared CSV in `data/` with fighter stats for the card. Output includes win probability and EV calculations against betting lines.

**gitignored artifacts**: `data/*.xlsx`, `models/*.joblib` — the source Excel training dataset and trained model files are not committed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ruckiryan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ruckiryan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
