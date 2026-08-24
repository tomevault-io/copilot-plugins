## energy-forecast

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Pipeline Commands

The pipeline is linear. Each step must complete before the next.

```bash
# Step 1 — Ingest raw data from APIs (~10–15 min for CAISO)
python -m src.data.ingest_all --market CAISO
python -m src.data.ingest_all --market ERCOT
python -m src.data.ingest_all --market both

# Step 2 — Build feature matrix
python -m src.data.feature_engineering --market CAISO

# Step 3 — Train all 6 models + evaluate
# CRITICAL: Set these env vars first on Intel Mac to avoid LSTM deadlock:
#   export MKL_THREADING_LAYER=GNU && export OMP_NUM_THREADS=1
python -m src.training.train_all_models --market CAISO
python -m src.training.train_all_models --market CAISO --skip-timegpt   # reuse saved TimeGPT preds

# Step 4a — Compute extended metrics (wMAPE, skill scores, arbitrage capture, breakdowns)
# Reads existing parquets — does NOT retrain. Outputs model_comparison.json.
export MKL_THREADING_LAYER=GNU && export OMP_NUM_THREADS=1
python -m src.evaluation.evaluate --market both

# Step 4b — Upload metrics + predictions to Supabase
# Run migrations/model_metrics_v2.sql in Supabase SQL editor first (one-time).
python -m src.deployment.upload_results --market both
python -m src.deployment.upload_results --market both --sample-rate 6   # every 6th row = ~4h resolution
python -m src.deployment.upload_results --market both --skip-predictions  # metrics only
python -m src.deployment.upload_results --market both --skip-metrics      # predictions only

# Frontend dev server
cd frontend && npm run dev
cd frontend && npm run build
```

## Architecture

**6-model benchmark pipeline** for hourly electricity LMP (Locational Marginal Price) forecasting in CAISO and ERCOT. Tests whether HMM regime detection improves a hybrid ML architecture vs. Nixtla's TimeGPT foundation model.

**Data flow:**
```
External APIs → data/raw/*.parquet → data/processed/*_features.parquet
→ data/models/{CAISO,ERCOT}/ (artifacts) + data/results/{CAISO,ERCOT}/ (predictions + JSON)
→ Supabase Postgres (model_metrics, predictions, model_registry tables)
→ React frontend on Vercel (project: lmp-model-leaderboard)
```

**The 6 models** (numbered as trained in `train_all_models.py`):
1. Naive baseline — time-of-day heuristic, no ML
2. TimeGPT zero-shot — Nixtla API
3. TimeGPT fine-tuned — Nixtla API, 30 finetune steps
4. XGBoost only — single model, no regime detection
5. HMM + XGBoost — 3 per-regime XGBoost models
6. HMM + XGBoost + LSTM — full hybrid (`HybridPipeline` class)

**Model architecture detail:**
- `RegimeDetector` (hmmlearn `GaussianHMM`) classifies each hour into one of 3 regimes based on LMP return, 24h rolling volatility, and LMP level
- `RegimeXGBoost` trains one XGBoost per regime; `get_residuals()` feeds the LSTM
- `RegimeLSTM` (2-layer PyTorch LSTM) predicts the next XGBoost residual from a 24h sliding window of past residuals — final forecast = XGBoost prediction + LSTM correction
- `HybridPipeline` orchestrates all three and also generates battery charge/discharge recommendations

**Temporal splits** (same for both markets):
- Train: up to 2024-03-31
- Val: 2024-04-01 → 2024-07-31
- Test: 2024-08-01 → end (held-out, never touched during training)

## Critical Constraints

**Timestamp handling:** All timestamps are normalized to **naive UTC** throughout the Python pipeline. The `to_naive_utc()` helper in `feature_engineering.py` handles both Series and DatetimeIndex. Never compare tz-aware with tz-naive; use `df["hour"]` (naive UTC) not `df["Time"]` (tz-aware) for splits and comparisons.

**HMM row alignment:** `prepare_observations()` calls `pct_change()` + `dropna()`, dropping the first ~24 rows. Downstream code must align via `df.iloc[-len(states):]`, not by integer position.

**CAISO LMP locations use hyphens:** `TH_NP15_GEN-APND`, not `TH_NP15_GEN_APND`.

**EIA API quirks:** Use respondent `"CISO"` for CAISO and `"ERCO"` for ERCOT. Append `T00` to start/end date strings for hourly frequency.

**ERCOT LMP source:** Uses `ErcotAPI.get_spp_day_ahead_hourly()` (not gridstatus's ERCOT class). Requires `ERCOT_API_USERNAME`, `ERCOT_API_PASSWORD`, `ERCOT_PUBLIC_API_SUBSCRIPTION_KEY` env vars. The `SPP` column is renamed to `LMP`.

**TimeGPT input format:** `_prepare_input()` uses `Location` as `unique_id` (one series per pricing node — 3 for CAISO, 4 for ERCOT). Collapsing to one `unique_id` causes duplicate timestamp errors.

**Per-regime XGBoost:** Strip `early_stopping_rounds` from params (the `REGIME_XGB_PARAMS` dict in `train_all_models.py`). Per-regime models have no `eval_set`, so early stopping is unavailable.

**`src/data/__init__.py`:** Imports only client classes (`LMPClient`, `EIAClient`, `WeatherClient`, `FREDClient`). Do NOT import `FeatureEngineer` or `ingest_all` there — causes circular import when running as `-m` module.

**Supabase:** The service key (`SUPABASE_SERVICE_KEY`) is used only in Python backend scripts. The React frontend uses the anon key (`VITE_SUPABASE_ANON_KEY`) exclusively. Never put the service key in frontend code.

## Key Files

| File | Role |
|------|------|
| `src/utils/config.py` | Single source of truth for all paths, hyperparams, market configs, API keys |
| `src/training/train_all_models.py` | Main CLI — also defines `XGBoostOnlyModel` and `HMMXGBoostModel` classes inline |
| `src/models/hybrid_pipeline.py` | `HybridPipeline` — full HMM→XGBoost→LSTM orchestration + battery recommendation logic |
| `src/evaluation/metrics.py` | All metric functions: MAE, RMSE, wMAPE, skill_score, arbitrage_capture_pct, etc. |
| `src/evaluation/evaluate.py` | Computes extended metrics from existing parquets; outputs `model_comparison.json` |
| `src/deployment/upload_results.py` | Uploader — metrics (wMAPE, skill scores, arbitrage capture, per-node/regime rows) + predictions |
| `migrations/model_metrics_v2.sql` | One-time Supabase schema migration (run before first upload) |
| `frontend/src/hooks/useModelData.js` | All Supabase data fetching; `useModelMetricsV2` supports Overall/Node/Regime views |

## V2 Metrics Schema

The `model_metrics` table (post-migration) has these columns:

| Column | Type | Notes |
|--------|------|-------|
| `mae`, `rmse` | FLOAT | Kept from v1 |
| `wmape` | FLOAT | Weighted MAPE: `sum(|a-p|)/sum(|a|)`. Stored as decimal (0.05 = 5%). |
| `skill_score_mae` | FLOAT | `1 - (model_mae / naive_mae)`. 0 = same as naive, positive = better. |
| `skill_score_rmse` | FLOAT | Same as above using RMSE. |
| `arbitrage_capture_pct` | FLOAT | % of perfect-foresight battery spread captured. Already in % (80.5 = 80.5%). |
| `directional_accuracy` | FLOAT | Kept from v1 |
| `node` | TEXT | NULL for market-level rows |
| `regime` | INTEGER | **-1 = overall aggregate**, 0/1/2 = HMM regime. Default -1. |

**Row layout per upload:** one overall row (`node=NULL, regime=-1`), one per node (`node=X, regime=-1`), one per regime (`node=NULL, regime=0/1/2`).

**Frontend filtering** (client-side in `useModelMetricsV2`):
- Overall view: `r.regime === -1 && r.node === null`
- Node view: `r.regime === -1 && r.node === nodeName`
- Regime view: `r.regime === regimeId && r.node === null`

## Node Attachment (evaluate.py)

Model parquets have no `Location` column (except TimeGPT which uses `unique_id`). Node labels are attached by merging on `(timestamp, actual_lmp)` against the features parquet. This works because:
- Each model parquet has one row per (hour, node) — 3 rows/hour for CAISO, 3–4 for ERCOT
- Actual LMP values per node are nearly always unique at each timestamp (< 0.1% collision rate at 5 decimal places)
- ERCOT's `HB_HOUSTON` has no training data (data gap) so naive baseline uses cross-location HOD mean as fallback

## Environment

Python 3.12, NumPy 2.4.2 (PyTorch warns but works), macOS Intel. The `venv/` directory is in the repo root — activate with `source venv/bin/activate`.

Frontend: Node with npm. Env vars prefixed `VITE_` are exposed to the browser (Vite convention); `SUPABASE_*` (no VITE prefix) are Python-only.

**`.env.local` format:** Do NOT use quotes or spaces around `=`. Correct: `VITE_SUPABASE_URL=https://...` (Vite dotenv includes literal quote characters in the value if quotes are present).

## What Was Done (2026-04-09)

**ERCOT retrain + negative-price feature application (complete):**

Ran the full pipeline for both markets with all bug fixes and 4 new features active:
1. Re-ran `feature_engineering --market both` — CAISO now 62 cols, ERCOT 60 cols
2. Retrained both markets (`--skip-timegpt`) — ERCOT now has post-fix artifacts for the first time
3. Ran `evaluate --market both` and `upload_results --market both` — Supabase fully updated

**Current best test-set results (post negative-price features):**

CAISO:
| Model | MAE | RMSE | Dir.Acc |
|-------|-----|------|---------|
| XGB Only | $1.13 | $4.81 | 94.0% |
| Hybrid Full | $1.18 | $4.93 | 93.1% |
| HMM+XGB | $1.24 | $6.35 | 93.7% |

ERCOT:
| Model | MAE | RMSE | Dir.Acc |
|-------|-----|------|---------|
| XGB Only | $4.62 | $15.43 | 85.5% |
| HMM+XGB | $5.13 | $17.52 | 84.4% |
| Hybrid Full | $5.46 | $18.12 | 80.5% |

Negative-price features gave modest ERCOT RMSE improvement for XGB Only (15.43 vs 15.99 pre-features). CAISO metrics roughly unchanged. XGB Only remains the best model on both markets — regime detection continues to hurt.

**Supabase upload:** 42 metric rows + ~55k prediction rows for CAISO; 48 metric rows + ~60k prediction rows for ERCOT. All current.

## What Was Done (2026-04-06)

**Hybrid model bug fixes (4 bugs, all in priority 1):**

The `hybrid_full` model was producing predictions equal to the prior row's actual LMP (~57% of rows), making its metrics meaningless. Four compounding bugs were found and fixed:

1. **Normalization bug** (`lstm_model.py:64`) — `fit()` used wrong operator precedence: `residuals - mean/std` instead of `(residuals - mean) / std`. LSTM trained on wrong scale; inference used correct scale → train/inference mismatch.
2. **Off-by-one context leak** (`hybrid_pipeline.py:predict()`) — context window passed to `get_residuals` included the current test row's actual LMP, so LSTM predicted the correction for row `i+1` and applied it to row `i`. Fixed by using `df.iloc[:-1]` for residual computation.
3. **Multi-node interleaving** (`hybrid_pipeline.py:train()` and `predict()`) — residuals were computed over all nodes interleaved (NP15/SP15/ZP26 mixed), so LSTM learned to predict the next node's residual rather than the next timestep's. Fixed by computing residuals per-node separately in training (concatenated), and filtering to the current node at inference.
4. **HMM alignment in `get_residuals`** (`xgboost_model.py`) — `prepare_observations` drops ~24 leading rows (pct_change + rolling std), so `regime_states` was shorter than `df`. The boolean mask raised a length mismatch exception, silently caught by the try/except in `_predict_hybrid_on_test`, which then fell back to `context["LMP"].iloc[-2]` — producing the exact lag-1 leakage pattern. Fixed by aligning `df = df.iloc[-len(regime_states):]` before masking.

**Result after fixes (CAISO test set):**
| Model | MAE before | MAE after | RMSE before | RMSE after | Dir.Acc before | Dir.Acc after |
|-------|-----------|-----------|-------------|------------|---------------|--------------|
| XGB Only | 0.93 | 1.11 | 4.68 | 4.89 | 94.1% | 94.1% |
| HMM+XGB | 1.17 | 1.22 | 9.24 | 6.42 | 94.0% | 94.0% |
| Hybrid Full | 6.14 | **1.14** | 14.48 | **4.83** | 35.5% | **93.3%** |

Hybrid now **beats XGB Only on RMSE** (4.83 vs 4.89). The exception handler was also updated to print warnings rather than silently falling back.

## What Was Done (2026-04-02)

**Dashboard metrics bug fix:**
- Deleted 15 stale v1 metric rows from Supabase (`wmape IS NULL`) that were shadowing v2 rows for the 5 ML models. Root cause: `upload_results.py` (v1) omitted `run_date`, so Postgres defaulted to `NOW()` — making v1 rows appear newer than v2 rows (which used midnight UTC). Naive baseline was unaffected because it was absent from the v1 `model_comparison.json`.
- Fixed `upload_results.py` to use `ignore_duplicates=True` so same-day re-runs no longer crash on the unique constraint.

**File consolidation:**
- Merged `upload_results_v2.py` into `upload_results.py` (single uploader for both metrics and predictions). Added `--skip-metrics` and `--skip-predictions` flags.
- Renamed `evaluate_v2.py` → `evaluate.py`; output renamed from `model_comparison_v2.json` → `model_comparison.json`. Deleted stale `model_comparison_v2.json` files from `data/results/`.

**Chart performance (PredictionOverlay):**
- Downsamples fetched predictions to ≤1000 points via uniform stride (was ~10k+ rows).
- Filters to a single node before downsampling (predictions table has 3 rows/hour for CAISO).
- Disabled Recharts line and tooltip animations (`isAnimationActive={false}`).
- Memoized tick/label formatters with `useCallback` to avoid re-creating `new Date()` on every mousemove.

**UI redesign:**
- Stripped Vite default template CSS from `index.css` — the `#root` rule had `width: 1126px; margin: 0 auto; text-align: center` which was capping and centering everything.
- Layout is now full-width: leaderboard table + bar chart share the top row (50/50 grid); prediction overlay spans full width below.
- Cards: `bg-gray-900 border border-gray-800 rounded-xl` with consistent padding.
- Sticky header with gradient title text and pill-style market selector.
- Regime view labels changed from R0/R1/R2 → Low Vol / Normal / High Vol.
- Tab title changed to "LMP Forecast"; Vite favicon removed.

## What's Left To Do

### Remaining accuracy improvement priorities (from error analysis)

- [ ] **Priority 3: Midnight hour sub-model** — hours 0–3 UTC account for all top-20 worst predictions in both markets. Consider hour-stratified ensemble or separate model for the overnight window (CAISO hour 22 worst: MAE $11.53; ERCOT hour 0 worst: MAE $17.03).
- [ ] **Priority 4: ERCOT spike prediction** — top 5% of hours generate 25% of ERCOT error (XGB MAE $29.74 on spikes vs $0.67 non-spike). CAISO spikes are better handled (XGB MAE $5.92). Consider spike-aware loss function or post-hoc spike classifier.
- [ ] **Priority 5: HMM regime classifier** — regime detection hurts both markets vs XGB Only (CAISO: +26% worse MAE, ERCOT: +10%). Regime 1 (volatile/spike hours) has the highest error and covers extreme prices. Consider retuning HMM or using a different regime signal.

### Frontend / deployment

- [ ] **Deploy frontend to Vercel** — `npm run build` passes; push to trigger Vercel redeploy
- [ ] **PredictionOverlay node selector** — chart currently locks to the first node; could add a dropdown so users can switch between pricing nodes
- [ ] **PredictionOverlay regime filter** — could shade background by HMM regime or let users filter to a single regime to see how predictions differ
- [ ] **Arbitrage capture nulls in regime view** — `arbitrage_capture_pct` is always `null` in per-regime rows because a battery operates across a full day, not within one regime; consider replacing with peak-hour MAE or hiding the column in regime view
- [ ] **ERCOT naive baseline quality** — `HB_HOUSTON` has no training data so its naive predictions use cross-market HOD mean, which is a rougher estimate than per-node means used for other hubs

---
> Source: [chaykup/energy-forecast](https://github.com/chaykup/energy-forecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
