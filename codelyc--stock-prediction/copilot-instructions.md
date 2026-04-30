## stock-prediction

> This repository is a PyTorch-based stock price prediction project. It covers data

# Agent Context

This repository is a PyTorch-based stock price prediction project. It covers data
collection, preprocessing, feature engineering, model training, and inference.
The workflow supports multiple model families (LSTM, Transformer, hybrid/PTFT,
diffusion, graph) and produces metrics and prediction charts.

## Quick Workflow
1. Fetch market data into `stock_daily/`.
2. Preprocess into `pkl_handle/train.pkl`.
3. Train a model, saving weights and normalization params in `models/`.
4. Run inference to generate metrics in `output/` and charts in `png/`.

## Domain & Data Conventions
- Symbols are normalized A-share style `ts_code` (6-digit, zero-padded, suffix removed).
- Dates are normalized to `YYYYMMDD` strings under `trade_date`.
- Raw daily CSV schema (from Akshare) is:
  `ts_code, trade_date, open, high, low, close, change, pct_change, vol, amount, amplitude, exchange_rate`.
- `common.import_csv` enriches CSVs with technical indicators (MACD, KDJ, BOLL, CCI, DMI, ATR, etc.)
  using helpers in `target.py`.
- `data_preprocess.py` serializes a `queue.Queue` of per-symbol DataFrames into `pkl_handle/train.pkl`.
  `ensure_queue_compatibility()` patches legacy queues for newer Python versions.

## Feature Engineering & Labels (Mental Model)
- `FeatureEngineer.transform()` is the first step for any DataFrame:
  - return/difference features (`target_mode`, `return_kind`, `difference_order`)
  - rolling stats on volatility columns (`sliding_windows`)
  - external CSV joins (macro/industry) keyed by `trade_date` with ffill
  - optional per-symbol normalization and `_symbol_index` for embeddings
- `Stock_Data` (Dataset in `common.py`) then:
  - keeps numeric columns only; drops `ts_code` and `_symbol_index`
  - caps feature count to `INPUT_DIMENSION` (default 30), truncating extras
  - builds sequences of length `SEQ_LEN` (default 5)
  - labels are derived from `use_list` in `init.py` (default 4 outputs: OHLC)
  - `predict_days > 0` yields multi-step labels; otherwise single-step

## Normalization & Checkpoints
- Training computes per-feature mean/std; inference loads them from `*_norm_params*.json`.
- If norm params are missing, `save_model()` can auto-compute from `pkl_handle/train.pkl`.
- Per-symbol stats may be stored in `symbol_norm_map` and reused for test/predict.
- Model init args are saved as `*_Model_args.json` alongside weights/optimizers.

## Config Layers & Global State
- `config.py` defines canonical paths; `init.py` defines global constants/queues
  (e.g., `SEQ_LEN`, `INPUT_DIMENSION`, `OUTPUT_DIMENSION`).
- `AppConfig` loads `config/config.yaml` (plus env overrides) for training + features.
- Many modules import from `init.py`, so global lists/queues are shared across modules.

## Model Interface Notes
- Models should accept `(inputs, symbol_index=None)` when symbol embedding is enabled.
- Training uses `Trainer` (`trainer.py`) with optional LR scheduler and early stopping.
- Hybrid/PTFT models use `HybridLoss` and its weighted components (see CLI args in `train.py`).

## Entry Points (CLI Wrappers)
- `scripts/getdata.py` -> `src/stock_prediction/getdata.py`
- `scripts/data_preprocess.py` -> `src/stock_prediction/data_preprocess.py`
- `scripts/train.py` -> `src/stock_prediction/train.py`
- `scripts/predict.py` -> `src/stock_prediction/predict.py`

Typical commands (uses `uv`):
```bash
uv sync
uv run python scripts/getdata.py --code 000001.SZ
uv run python scripts/data_preprocess.py --pklname train.pkl
uv run python scripts/train.py --mode train --model transformer --epoch 2
uv run python scripts/predict.py --model transformer --test_code 000001 --predict_days 3
```

## Key Code Areas
- `src/stock_prediction/feature_engineering.py`: feature pipeline and indicators.
- `src/stock_prediction/trainer.py`: training loop, early stopping, metrics.
- `src/models/`: model implementations and registrations.
- `src/stock_prediction/metrics.py`: evaluation metrics (RMSE, MAPE, VaR, etc.).

## Configuration & Artifacts
- `config/config.yaml`: global settings (batch size, epochs, feature config).
  - `features.external_sources` includes an optional macro CSV path; the default
    path points to `config/external/macro_sample.csv` (not present by default).
- `pkl_handle/train.pkl`: preprocessed training data.
- `models/`: saved weights and `*_norm_params*.json`.
- `output/metrics_*.json`: evaluation summaries.
- `png/`: prediction plots.

## Tests & Quality
- Tests live in `tests/` and run with `uv run pytest -q`.
- If adding a new model, register it under `src/models/` and
  update docs/tests accordingly.

## Docs
- `docs/system_design.md`: architecture decisions.
- `docs/model_strategy.md`: model choices and recommendations.
- `docs/user_guide.md`: CLI/module usage.
- `docs/maintenance.md`: structural changes and fixes.

---
> Source: [codelyc/stock-prediction](https://github.com/codelyc/stock-prediction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
