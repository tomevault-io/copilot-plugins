## season

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a production-ready **category-level sales forecasting pipeline** for retail/fresh products. It uses MSTL decomposition for seasonality extraction, temperature-modulated yearly patterns, and LightGBM for final predictions. The system is designed to be minimal, configurable via `config.yaml`, and suitable for categories with 10+ SKUs.

**Core Philosophy**: Simplicity over complexity. Mathematical modulation instead of ML models where possible. All parameters user-configurable.

## Common Commands

```bash
# Initial setup
pip install -r requirements.txt
python demo_synthetic_data.py    # Generate demo data

# Main workflow (in order)
python run.py                     # Run full forecasting pipeline
python validate.py                # Time series cross-validation (4 folds)
python visualize_forecast.py      # Generate all plots

# One-time hyperparameter tuning (optional but recommended)
python tune_hyperparameters.py    # Saves to best_hyperparameters.yaml
# After tuning, run.py and validate.py automatically use tuned params

# Individual visualizations (if needed separately)
# All plots are now in visualize_forecast.py - no separate scripts
```

## Architecture Overview

### Pipeline Flow (forecaster.py)

The `CategoryForecaster` class orchestrates 7 sequential steps:

```
SKU-Day Data
    ↓
1. aggregate_to_category()      # Filter categories, interpolate stockout sales (SKU or category level)
    ↓
2. decompose()                   # MSTL: extract trend + weekly + yearly seasonality (on training data)
    ↓
3. forecast_trend()              # Exponential smoothing on trend component
    ↓
4. prepare_features()            # Engineer lags, rolling stats, cyclical encodings
    ↓
5. train_seasonal_model()        # Calculate temperature sensitivity (math formula, NO model training)
    ↓
6. train_lgbm()                  # Train LightGBM on total sales (using components as features)
    ↓
7. generate_forecast()           # Combine components + LightGBM prediction (uses last_year or average strategy)
```

### Key Design Decisions

**1. Temperature Modulation (NOT a trained model)**
- Located in `train_seasonal_model()` (lines 231-244)
- Formula: `seasonal_yearly × (1 + temp_sensitivity × temp_deviation)`
- Bounded to ±30% adjustment (0.7-1.3 multiplier)
- Configuration: `decomposition.temp_sensitivity` (default 0.02)
- **Important**: This is pure math - no model training, no overfitting risk

**2. Data Leakage Prevention**
- Rolling features use `shift(1)` AFTER rolling: `df['sales'].rolling(7).mean().shift(1)`
- This gives Day 100 the rolling mean of Days 93-99 (most recent without leakage)
- Previous approach used `shift(30)` before rolling, throwing away 27 days of information
- Forecasting uses fixed historical values (lines 405-415), so no recursive issues

**3. Automatic Hyperparameter Loading**
- `train_lgbm()` checks if `best_hyperparameters.yaml` exists
- If found: loads tuned params automatically
- If not: falls back to `config.yaml` defaults
- **One-time tuning, zero manual copying**

**4. Preprocessing Quality Controls**
- `min_skus_per_category`: Filter noisy categories with insufficient SKUs
- `instock_threshold`: Linear interpolation when instock < 85% (estimates true demand during stockouts)
- `interpolation_level`: 'sku' (default, recommended) or 'category' - where to apply stockout correction
- `instock_rate` is NOT used as a model feature (prevents data leakage)

**5. Yearly Seasonality Strategy**
- `yearly_seasonality_strategy`: 'last_year' (default) or 'average'
- **'last_year'**: Uses only the most recent 365 days of seasonal pattern (recommended for dynamic markets)
- **'average'**: Averages across all historical years (better for stable markets with high substitution)
- Configuration: `decomposition.yearly_seasonality_strategy` in config.yaml
- Applied during `generate_forecast()` when mapping day-of-year to seasonal values

### File Roles

**Core Pipeline**:
- `forecaster.py`: The CategoryForecaster class with all 7 pipeline steps
- `config.yaml`: Single source of truth for ALL parameters
- `config_loader.py`: Dot-notation config accessor (`config.get('preprocessing.min_skus_per_category')`)

**Execution Scripts**:
- `run.py`: Main pipeline - calls all 7 steps sequentially
- `validate.py`: Time series cross-validation with fixed-size moving window
- `tune_hyperparameters.py`: Optuna-based hyperparameter search (LightGBM only)
- `visualize_forecast.py`: All visualizations (forecast analysis, weekly patterns, feature importance)
- `demo_synthetic_data.py`: Generate synthetic data for testing

**Auto-Generated Files**:
- `best_hyperparameters.yaml`: Tuned LightGBM params (auto-loaded by forecaster.py)
- `category_day_aggregated.csv`: Aggregated data with preprocessing applied
- `pooled_training_data.csv`: Engineered features for training
- `category_forecasts_30day.csv`: Final forecasts
- `validation_metrics.csv`: Performance metrics by fold/category
- `validation_predictions.csv`: Actuals vs forecasts for each fold

## Configuration Guide

All configuration is in `config.yaml`. Key sections:

**Preprocessing** (affects data quality):
- `min_skus_per_category`: 5-20 depending on retailer size
- `instock_threshold`: 0.80-0.95 (higher = more sensitive to stockouts)
- `interpolation_level`: 'sku' (default, granular) or 'category' (high substitution scenarios)

**Decomposition** (affects seasonality):
- `temp_sensitivity`: 0.01 (shelf-stable) to 0.05 (ice cream, seasonal produce)
- `yearly_seasonality_strategy`: 'last_year' (default, recent patterns) or 'average' (stable long-term)

**Trend** (affects trend forecasting):
- `smoothing_level`: Controls level smoothing (0-1)
- `smoothing_trend`: Controls trend smoothing (0-1)
- `damping_trend`: Prevents over-extrapolation (0.8-0.98)

**Features** (affects model inputs):
- `lag_days`: Recent lags are most important (1, 7, 14, 28)
- `rolling_windows`: Captures trends (7, 14, 28)
- `min_non_nan_pct`: Minimum non-null percentage to include feature

**LightGBM** (auto-overridden by tuned params):
- Default values in config.yaml
- After running `tune_hyperparameters.py`, these are ignored
- To revert to defaults: delete `best_hyperparameters.yaml`

**Validation**:
- `n_splits`: Number of CV folds (default: 30 for full seasonal coverage)
- `train_window_days`: Fixed training window size (default: 800 days)

**Visualization**:
- `lookback_days`: Historical days to show before test period (default: 180)

## Validation Methodology

**Time Series Cross-Validation** (validate.py):
- **Fixed-size moving window** approach (each fold has same 730-day training window)
- 3 folds (configurable), 7-day forecast horizon
- Each fold trains on exactly 730 days (2 years), window slides forward for next fold
- **Ensures yearly seasonality**: All folds have >= 730 days for MSTL decomposition
- Uses SAME `generate_forecast()` method as production (lines 84-113)
- **No test data leakage**: Model sees only training data when generating forecasts
- **Expected inputs available**: Temperature (weather forecasts) and promotions (planned schedule)
- **WAPE is primary metric**: Weighted by sales volume (more reliable than MAPE)

Current performance: ~11.2% WAPE after tuning

## Feature Importance

Top features (by LightGBM gain):
1. `sales_lag_7`: Most recent weekly sales
2. `seasonal_weekly`: Day-of-week pattern
3. `sales_lag_14`: Two-week sales
4. `seasonal_yearly`: Annual seasonality
5. `price_ratio`: Price changes vs 28-day average

Recent sales history dominates, validating the `shift(1)` after rolling approach.

## Data Requirements

**Input CSV** (`demo_sku_data.csv`):
```
Required columns: sku_id, category, date, sales, instock_rate, price,
                 temperature, holiday_gap, main_promo, other_promo
Minimum data: 2 years daily records (730 days)
Minimum SKUs: 10 per category (configurable)
```

**Data Quality**:
- No missing dates (daily continuity required for MSTL)
- Complete temperature data (needed for modulation)
- Instock rate 0-100 (percentage, not 0-1)

## Troubleshooting

**"No categories have >= N SKUs"**
- Lower `preprocessing.min_skus_per_category` in config.yaml

**"MSTL decomposition failed"**
- Need at least 2 years of data (730 days)
- Check for missing dates in input

**High WAPE (>20%)**
- Run `tune_hyperparameters.py` for better LightGBM params
- Adjust `temp_sensitivity` for product type
- Increase `min_skus_per_category` to filter noise

**Validation slower than expected**
- Reduce `validation.n_splits` from 4 to 3
- Reduce `optuna.n_trials` from 50 to 30

## Important Implementation Notes

1. **Never remove `shift(1)` from rolling features** - this was carefully optimized to maximize recency while preventing data leakage

2. **Temperature modulation is NOT a model** - it's a mathematical formula. Don't try to "train" it or add complexity.

3. **Validation metrics reflect production performance** - the validation uses the same non-recursive forecasting approach as `generate_forecast()`, so WAPE estimates are realistic.

4. **Auto-tuning is one-time** - Once `best_hyperparameters.yaml` exists, all scripts use it automatically. Delete to revert to defaults.

5. **Feature engineering happens in `prepare_features()`** - This is where lags, rolling windows, and cyclical encodings are created. All feature logic is centralized here.

6. **Forecasting is non-recursive** - `generate_forecast()` uses fixed historical values for all lag/rolling features. This simplifies logic and matches validation approach.

## Recent Improvements (2025-11)

The following enhancements were made to improve forecast accuracy and production readiness:

### 1. SKU-Level Interpolation (forecaster.py:95-120, config.yaml:26)
**Addition**: `interpolation_level: 'sku'` parameter for stockout correction
**Options**:
- **'sku' (default)**: Interpolates each SKU individually before aggregation
- **'category'**: Interpolates at category level after aggregation (original behavior)

**Rationale**:
- SKU-level captures true demand without signal loss from partial assortment stockouts
- Better handles mixed substitution scenarios (some SKUs substitute, others don't)
- Prevents category-level averaging from masking individual SKU stockout patterns

**When to use**:
- **'sku'**: Most retail scenarios, fresh products, unique SKUs, mixed substitution
- **'category'**: Pure commodities with perfect substitution, legacy compatibility

**Implementation**: Two new methods `_interpolate_sku_level()` and `_interpolate_category_level()`

### 2. Configurable Yearly Seasonality Strategy (forecaster.py:366-393, config.yaml:44)
**Addition**: `yearly_seasonality_strategy: 'last_year'` parameter
**Options**:
- **'last_year' (default)**: Uses only most recent 365 days of yearly seasonal pattern
- **'average'**: Averages across all historical years (original behavior)

**Rationale**:
- Recent year typically more relevant for forecasting (business changes, customer preferences)
- Easier to explain to stakeholders: "next year follows last year's pattern"
- More responsive to market trends while still temperature-modulated

**When to use**:
- **'last_year'**: Dynamic markets, evolving preferences, recent business changes
- **'average'**: Stable categories, filtering out anomalies, long-term patterns

**Implementation**: Reads config in `generate_forecast()`, creates day-of-year mapping from either last year or all years

### 3. Fixed MSTL Seasonality Logic (forecaster.py:125-133)
**Issue**: When MSTL returned 1D output with 730+ days, code incorrectly assumed yearly-only seasonality
**Fix**: Now correctly treats 1D output as weekly seasonality (yearly extraction failed)
**Rationale**: Weekly patterns are more fundamental; if yearly is weak/noisy, MSTL extracts only weekly

### 2. Added Damped Trend Forecasting (forecaster.py:147-163, config.yaml:45)
**Addition**: `damping_trend: 0.95` parameter for exponential smoothing
**Benefit**: Prevents unrealistic trend extrapolation for 7-30 day forecasts
**Behavior**: Trend flattens over horizon instead of continuing linearly
**Configuration**: 0.8 (strong damping) to 0.98 (light damping), default 0.95

### 3. Added lag_1 Feature (config.yaml:51)
**Addition**: `lag_days: [1, 7, 14, 28]` - now includes yesterday's sales
**Impact**: Yesterday's sales is typically the most predictive feature
**Expected**: `sales_lag_1` likely to become top feature by importance
**Previous oversight**: Only had weekly+ lags, missing most recent information

### 4. Future Inputs from CSV Files (config.yaml:100-105, run.py:12-169)
**New capabilities**:
- `future_prices.csv` - Planned pricing schedules
- `future_promos.csv` - Promotional calendars
- `future_temperature.csv` - Weather forecasts

**Behavior**:
- If files exist: Uses provided data
- If missing: Falls back to defaults (last price, no promos, historical temp avg)
- Templates provided: `*_template.csv` files show required format

**Benefits**:
- More accurate forecasts with planned business inputs
- Production-ready: integrates with pricing/promo systems
- Optional: backward compatible with automatic fallbacks

### 5. Sales-Weighted Metrics (validate.py:42-61, 166-180)
**Change**: Overall metrics now weighted by sales volume instead of simple averaging
**Rationale**: High-volume categories should influence metrics more than low-volume
**Example**: 10,000-unit category has 10× more weight than 1,000-unit category
**Output**: Shows `Total Sales: X` along with weighted metrics
**Consistency**: Category breakdown also shows `Total_Sales` column

### 6. Sales-Weighted Hyperparameter Tuning (tune_hyperparameters.py:67-132)
**Change**: Optuna objective now uses sales-weighted MAPE across folds
**Previous**: Simple average MAPE (all folds equal weight)
**New**: Folds with higher sales volume have more influence
**Benefit**: Hyperparameters optimized for high-volume performance (business impact)
**Consistency**: Matches validate.py's weighted metric approach

### 7. Full Pipeline Hyperparameter Tuning (tune_hyperparameters.py:49-123)
**Major refactor**: Tuning now uses complete forecasting pipeline per trial
**Previous**: Direct LightGBM training on pre-computed features (pooled_training_data.csv)
**New**: Each trial runs:
  1. MSTL decomposition
  2. Trend forecasting (with damped trend)
  3. Feature engineering
  4. Temperature modulation
  5. LightGBM training with trial params
  6. `generate_forecast()` with actual test inputs (temps, promos, prices)

**Benefits**:
- Hyperparameters optimized for actual forecasting methodology
- Consistent with validate.py (same evaluation approach)
- Realistic performance estimates

**Trade-offs**:
- Slower (~5-10× more time per trial)
- More accurate (tunes for production behavior, not just LightGBM)

### 8. Tuning with Full Seasonal Coverage (tune_hyperparameters.py:24-82)
**Change**: Hyperparameter tuning now uses same n_splits as validation
**Previous**: Hardcoded to 3 CV splits regardless of config
**New**: Reads `validation.n_splits` from config.yaml (default: 30 splits)

**Rationale**:
- For seasonal categories, 3 splits may only capture one seasonal phase (e.g., all "ramp up" or all "peak season")
- Hyperparameters tuned on limited seasonal variation may not generalize
- 30 splits ensures tuning across multiple seasonal cycles and phases
- Matches validate.py methodology for consistency

**Benefits**:
- Hyperparameters robust across full seasonal cycle
- Prevents overfitting to specific seasonal phases
- Same evaluation methodology as production validation

**Trade-offs**:
- Significantly longer tuning time (e.g., 50 trials × 30 folds = 1,500 pipeline runs vs 150)
- More realistic performance estimates
- Can reduce n_splits in config if faster exploratory tuning needed

**Implementation**: Method now reads `self.config.get('validation.n_splits')` with validation checks

### 9. Improved .gitignore
**Additions**: Comprehensive exclusions for output files, virtual envs, IDEs
**Keeps**: Template CSVs (`*_template.csv`) for user reference
**Excludes**: Generated CSVs, PNGs, best_hyperparameters.yaml (regenerated)

### 10. Configuration Cleanup (2025-11)
**Removed unused parameters from config.yaml**:
- **`data` section** (n_categories, n_skus_per_category, n_days, start_date): Only used by demo_synthetic_data.py utility, which has its own defaults
- **`trend.use_exponential_smoothing`**: Was never read; exponential smoothing always active
- **`visualization.dpi`** and **`visualization.figsize_per_category`**: Were hardcoded in visualize_forecast.py

**Made configurable**:
- **`visualization.lookback_days`**: Now properly read from config.yaml (default: 180) instead of being hardcoded

**Rationale**: Removes configuration dead code and ensures all visualization parameters are truly configurable

### 11. Day-of-Month Features (2025-11)
**Addition**: Three new temporal features in `prepare_features()` (forecaster.py:234-238, 394-398)
- **`day_of_month`**: Numeric day (1-31)
- **`is_month_start`**: Binary flag for days 1-7 (payday effect - first week of month)
- **`is_month_end`**: Binary flag for days 24-31 (end-of-month shopping surge)

**Rationale**:
- Captures monthly shopping patterns driven by salary cycles
- First week (payday): Increased spending on groceries and essentials
- Last week (pre-payday): Budget constraints or stocking up before next cycle
- Particularly important for fresh products and high-involvement categories

**Expected impact**: 0.5-1.5pp WAPE improvement, especially for categories with strong monthly cycles

**Implementation**: Added to both training (`prepare_features()`) and forecasting (`generate_forecast()`) pipelines for consistency

## Updated Feature Importance (After Recent Additions)

Expected top features after improvements:
1. **`sales_lag_1`**: Yesterday's sales (most recent)
2. `sales_lag_7`: Last week's sales
3. `seasonal_weekly`: Day-of-week pattern
4. `sales_lag_14`: Two weeks ago
5. `seasonal_yearly`: Annual seasonality (temp-modulated)
6. **`is_month_start`** or **`is_month_end`**: Payday/end-of-month effects (NEW)
7. `price_ratio`: Price changes vs 28-day average

**Note**: `day_of_month` may rank high if monthly patterns are strong; `is_month_start`/`is_month_end` provide binary encoding for easier model interpretation

## Configuration Summary (Post-Improvements)

All new features are in `config.yaml`:
```yaml
# Preprocessing (NEW: SKU-level interpolation)
preprocessing:
  min_skus_per_category: 10
  instock_threshold: 0.85
  interpolation_level: 'sku'  # NEW: 'sku' (default) or 'category'

# Decomposition (NEW: yearly seasonality strategy)
decomposition:
  temp_sensitivity: 0.02
  yearly_seasonality_strategy: 'last_year'  # NEW: 'last_year' (default) or 'average'

# Trend forecasting (NEW: damped trend)
trend:
  smoothing_level: 0.8
  smoothing_trend: 0.2
  damping_trend: 0.95  # NEW: prevents over-extrapolation

# Features (NEW: lag_1 added)
features:
  lag_days: [1, 7, 14, 28]  # NEW: includes yesterday

# Input files (NEW: optional CSV inputs)
input:
  sku_data: "demo_sku_data.csv"
  future_prices: "future_prices.csv"      # NEW: optional
  future_promos: "future_promos.csv"      # NEW: optional
  future_temperature: "future_temperature.csv"  # NEW: optional

# Visualization
visualization:
  lookback_days: 180  # Days of history to show before forecast
```

## Validation Consistency

All three scripts now use the same methodology:
- **run.py**: Production forecasting with optional CSV inputs
- **validate.py**: Time series CV with actual test data inputs + weighted metrics + configurable n_splits
- **tune_hyperparameters.py**: Full pipeline per trial + weighted objective + **same n_splits as validate.py**

**Key consistency**: Both validate.py and tune_hyperparameters.py read `validation.n_splits` from config.yaml, ensuring hyperparameters are tuned on the same seasonal coverage as final validation. This prevents tuning bias from limited seasonal phases.

This ensures tuned hyperparameters are optimal for production forecasting across all seasonal conditions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kamalyunus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kamalyunus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
