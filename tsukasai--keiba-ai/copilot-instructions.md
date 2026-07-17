## keiba-ai

> This is a Japanese horse racing (Keiba) AI prediction system for JRA (Japan Racing Association) races. The goal is to predict Exacta (1st and 2nd place in exact order) outcomes and maximize ROI using expected value-based betting strategy.

# Claude Code Instructions

## Project Overview

This is a Japanese horse racing (Keiba) AI prediction system for JRA (Japan Racing Association) races. The goal is to predict Exacta (1st and 2nd place in exact order) outcomes and maximize ROI using expected value-based betting strategy.

## Tech Stack

- **Analysis/ML Training**: Python (use `uv` for package management)
- **Inference & Live Prediction**: Rust (single binary, no Python dependency)
- **Data Source**: Kaggle JRA Dataset (2019-2021) + JRA official (future: JRA-VAN paid data)

## Project Structure

```
keiba-ai/
├── config/settings.py        # Configuration
├── data/
│   ├── raw/                  # Raw data (Kaggle CSV files)
│   ├── processed/            # Processed/feature-engineered data
│   └── cache/                # Scraper cache (auto-created)
├── src/
│   ├── data_collection/      # Data download scripts
│   ├── preprocessing/        # Feature engineering
│   ├── models/               # ML models, backtesting, calibration
│   └── api/                  # Rust inference API & CLI
│       ├── src/
│       │   ├── main.rs       # Entry point (CLI + server)
│       │   ├── cli.rs        # CLI commands (serve, predict, backtest, live)
│       │   ├── routes.rs     # API handlers
│       │   ├── model.rs      # ONNX inference
│       │   ├── backtest.rs   # Walk-forward backtesting
│       │   ├── exacta.rs     # Exacta probability (Harville formula)
│       │   ├── trifecta.rs   # Trifecta probability
│       │   ├── quinella.rs   # Quinella probability
│       │   ├── trio.rs       # Trio probability
│       │   ├── wide.rs       # Wide probability
│       │   ├── betting.rs    # EV, Kelly criterion
│       │   ├── calibration.rs # Probability calibration
│       │   ├── config.rs     # Configuration
│       │   ├── types.rs      # Request/response types
│       │   ├── scraper/      # Live race scraper (Rust)
│       │   │   ├── mod.rs           # Module definition
│       │   │   ├── browser.rs       # chromiumoxide browser automation
│       │   │   ├── cache.rs         # File-based cache with TTL
│       │   │   ├── rate_limiter.rs  # Token bucket rate limiter
│       │   │   ├── feature_builder.rs # 39 ML features
│       │   │   ├── parsers/         # HTML/JSON parsers
│       │   │   │   ├── race_card.rs # Race card parser
│       │   │   │   ├── horse.rs     # Horse profile parser
│       │   │   │   ├── jockey.rs    # Jockey profile parser
│       │   │   │   ├── trainer.rs   # Trainer profile parser
│       │   │   │   └── odds.rs      # Odds API parser
│       │   │   └── historical/      # Historical data scraper
│       │   │       ├── mod.rs       # URL builders
│       │   │       ├── race_list.rs # Race schedule parser
│       │   │       ├── race_result.rs # Race result parser
│       │   │       └── odds_history.rs # Historical odds parser
│       │   └── storage/         # SQLite storage layer
│       │       ├── mod.rs
│       │       ├── schema.rs    # Table definitions
│       │       └── repository.rs # CRUD operations
│       └── scripts/
│           └── prepare_backtest_data.py
├── data/
│   └── historical/
│       └── keiba.db             # SQLite database (scraped data)
├── scripts/                  # Python scripts
│   ├── retrain.py            # Model retraining pipeline
│   ├── run_validation.py     # Validation backtest
│   └── export_onnx.py        # ONNX model export
├── tests/                    # Python unit tests (213 tests)
│   └── test_*.py             # Model/backtesting tests
└── notebooks/                # Jupyter exploration
```

## Development Phases

All phases completed:

- [x] **Phase 1**: Data Collection & Exploration - Kaggle dataset (2019-2021)
- [x] **Phase 2**: Model Building - LightGBM position probability model
- [x] **Phase 3**: Backtesting - Walk-forward validation (+19.3% ROI with calibration, on optimistic post-race/estimated odds — see Known Issues; not verified on real combination odds)
- [x] **Phase 4**: Rust Inference API - REST API with all 5 bet types
- [x] **Phase 5**: Live Race Scraper - netkeiba.com integration
- [x] **Phase 6**: Full Rust Migration - Single binary with `live` command (no Python dependency)

## Data Source

### Primary: Kaggle JRA Horse Racing Dataset
- URL: https://www.kaggle.com/datasets/takamotoki/jra-horse-racing-dataset
- Period: 1986-2021 (use 2019-2021 for this project)
- Format: CSV (pre-processed, easy to use)
- Contents: Race results, betting odds, lap times, corner passing orders

### Historical: netkeiba.com Scraper
- URL: https://db.netkeiba.com/
- Period: 2022-2025 (scraped via `scrape-historical` command)
- Format: SQLite database (`data/historical/keiba.db`)
- Contents: Race results, **final win odds (単勝) per horse**, horse/jockey/trainer data
- **NOT available**: pre-race odds for all combinations. netkeiba does not retain
  full-combination odds after a race finishes — only winning-combination payouts
  remain. Real all-combination odds require JRA-VAN (paid) or forward-capture of
  live odds before post time.

### Future: JRA-VAN DataLab (Paid)
- For production-grade data with training/workout info
- Official JRA data with more features
- Requires Windows (JV-Link is ActiveX COM)

## Key Concepts

### Expected Value Strategy
```
expected_value = predicted_probability × odds
Buy only when expected_value > 1.0
```

### Supported Bet Types

| Bet Type | Japanese | Description | Combinations (18 horses) |
|----------|----------|-------------|--------------------------|
| Exacta | 馬単 | 1st-2nd in exact order | 306 |
| Trifecta | 三連単 | 1st-2nd-3rd in exact order | 4,896 |
| Quinella | 馬連 | 1st-2nd any order | 153 |
| Trio | 三連複 | 1st-2nd-3rd any order | 816 |
| Wide | ワイド | 2 horses both in top 3 | 153 |

All bet types use the Harville formula for probability calculation.

### Model Output Design
The model should output **probability distribution for each horse's finishing position**, not just win probability. This allows easy extension to other bet types.

```python
# Example output per horse
{
    "horse_1": {"1st": 0.15, "2nd": 0.12, "3rd": 0.10, ...},
    "horse_2": {"1st": 0.08, "2nd": 0.10, "3rd": 0.12, ...},
    ...
}
```

## Features to Consider

### Basic Features (Available in Kaggle data)
| Feature | Description | Japanese Term |
|---------|-------------|---------------|
| Horse info | Age, sex, weight | horse_age, horse_sex, horse_weight |
| Jockey | Jockey name, win rate | jockey |
| Trainer | Trainer name, win rate | trainer |
| Odds | Win odds, place odds | odds |
| Post position | Gate number (1-18) | gate_number, post_position |
| Distance | Race distance (1000-3600m) | distance |
| Surface | Turf or Dirt | turf/dirt |
| Track condition | Good/Yielding/Soft/Heavy | track_condition |
| Weight carried | Handicap weight | weight_carried |
| Past performance | Previous race results | past_performance |

### Advanced Features (Derive from data)
| Feature | Description | Calculation |
|---------|-------------|-------------|
| Running style | Front-runner/Stalker/Closer | From corner positions |
| Distance aptitude | Performance by distance | distance_aptitude |
| Surface aptitude | Turf vs Dirt performance | surface_aptitude |
| Track aptitude | Performance at specific tracks | track_aptitude |
| Class level | Race grade history | class_level |
| Form | Recent performance trend | form |

### Blood Features (Important in JRA)
| Feature | Description |
|---------|-------------|
| Sire | Father's lineage |
| Broodmare Sire | Mother's father |
| Sire line | Pedigree line |

**Note**: Training/workout data is NOT available in Kaggle dataset. Will be added when JRA-VAN is integrated.

## Commands

### Setup (using uv)
```bash
cd keiba-ai
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

### Download Kaggle Data
```bash
# Requires Kaggle API credentials (~/.kaggle/kaggle.json)
kaggle datasets download -d takamotoki/jra-horse-racing-dataset
unzip jra-horse-racing-dataset.zip -d data/raw/
```

### Phase 1 Execution
```bash
# 1. Download data (or manually from Kaggle website)
uv run python src/data_collection/download_kaggle.py

# 2. Explore data
uv run jupyter notebook notebooks/01_data_exploration.ipynb

# 3. Feature engineering
uv run python src/preprocessing/feature_engineering.py
```

### Run Rust CLI (Production)

```bash
# Build
cd src/api && cargo build --release

# Live prediction (main use case - single command, no Python needed)
./target/release/keiba-api live 202506050811
./target/release/keiba-api live 202506050811 --bet-type trifecta --ev-threshold 1.2 --verbose

# Start API server
./target/release/keiba-api serve --port 8080

# Run prediction from JSON file
./target/release/keiba-api predict race.json --bet-types all --format table

# Run backtest (from project root)
./target/release/keiba-api backtest \
  ./data/processed/backtest_features.parquet \
  --odds ./data/processed/exacta_odds.csv \
  --bet-type exacta \
  --calibration ./data/models/calibration.json \
  --ev-threshold 1.0

# CLI help
./target/release/keiba-api --help
./target/release/keiba-api live --help
```

### Model Retraining Pipeline

```bash
# Full pipeline: features → training → validation → export (LightGBM default)
python scripts/retrain.py

# Use different model types
python scripts/retrain.py --model-type catboost   # CatBoost (better for categorical features)
python scripts/retrain.py --model-type xgb        # XGBoost
python scripts/retrain.py --model-type ensemble   # Ensemble (LightGBM+XGBoost+CatBoost)

# Run hyperparameter optimization before training
python scripts/retrain.py --optimize              # 50 trials (default)
python scripts/retrain.py --optimize --n-trials 100  # More optimization trials

# Skip feature engineering (use existing features.parquet)
python scripts/retrain.py --skip-features

# Only run validation (requires trained model)
python scripts/retrain.py --validate-only

# Only export ONNX + calibration (requires trained model)
python scripts/retrain.py --export-only
```

#### Model Types

| Type | Description | Best For |
|------|-------------|----------|
| `lgbm` | LightGBM (default) | General purpose, fast training |
| `catboost` | CatBoost | Categorical features (jockey/trainer) |
| `xgb` | XGBoost | Alternative gradient boosting |
| `ensemble` | Weighted ensemble | Maximum accuracy, stable ROI |
| `ensemble --stacking` | Stacking ensemble | Meta-learner combines base models |

Outputs:
- `data/models/position_model_39features.pkl` - Pickled model
- `data/models/position_model_{type}.pkl` - Model by type
- `data/models/position_model.onnx` - ONNX model for Rust (LightGBM only)
- `data/models/calibration.json` - Fitted calibration config
- `data/models/best_params_{type}.json` - Optimized hyperparameters (if --optimize used)

#### CLI Commands

| Command | Description |
|---------|-------------|
| `live` | **Live prediction** - Scrape race data and predict (single command) |
| `serve` | Start REST API server |
| `predict` | Run prediction on race JSON file |
| `backtest` | Run walk-forward backtest on Kaggle data |
| `scrape-historical` | Scrape historical race data from db.netkeiba.com |
| `backtest-historical` | Run backtest on scraped historical data (SQLite) |
| `paper-record` | **Forward paper-trade** - record EV bets with real pre-race odds (before post time) |
| `paper-settle` | Settle pending paper bets against official payouts (after the race) |
| `paper-report` | Report verified ROI / hit-rate from settled paper bets |

#### Live Command Options

```bash
keiba-api live <RACE_ID> [OPTIONS]

Arguments:
  <RACE_ID>  Race ID (e.g., 202506050811)

Options:
  -b, --bet-type <BET_TYPE>      Bet type: exacta, trifecta [default: exacta]
      --ev-threshold <THRESHOLD>  EV threshold for recommendations [default: 1.0]
  -o, --output <FILE>            Output file path (JSON)
      --calibration <FILE>       Calibration config file [default: data/models/calibration.json]
  -f, --force                    Force refresh (ignore cache)
  -v, --verbose                  Show detailed progress
```

#### Race ID Format

`YYYYRRCCNNDD` where:
- `YYYY` = Year (e.g., 2025)
- `RR` = Racecourse code (06=Nakayama, 05=Tokyo, etc.)
- `CC` = Meeting number
- `NN` = Day number
- `DD` = Race number (01-12)

Example: `202506050811` = 2025 Nakayama 5th meeting 8th day Race 11 (有馬記念)

#### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| GET | `/model/info` | Model information |
| POST | `/predict` | Race prediction (all 5 bet types) |

#### Historical Data Commands

```bash
# Scrape historical race data from db.netkeiba.com (results + final win odds)
keiba-api scrape-historical --date 2024-12-22 --db data/historical/keiba.db
keiba-api scrape-historical --start 2022-01-01 --end 2025-12-31

# Backtest with scraped historical data
keiba-api backtest-historical --db data/historical/keiba.db \
    --start 2023-01-01 --end 2024-12-31 \
    --bet-type exacta --ev-threshold 1.0

# Python orchestrator (with resume capability)
python src/data_collection/historical_scraper.py --start 2024-01-01 --end 2024-12-31
python src/data_collection/historical_scraper.py --resume  # Continue from last date
```

Options for `scrape-historical`:
- `--date`: Single date to scrape (YYYY-MM-DD)
- `--start/--end`: Date range to scrape
- `--db`: SQLite database path
- `--force`: Re-scrape existing data
- `--verbose`: Show detailed progress

> **Removed**: `--include-odds` previously targeted `db.netkeiba.com/odds/.../umatan/`,
> which returns HTTP 404 — that endpoint does not exist, and netkeiba does not retain
> full-combination odds after a race. The flag never populated any data and was removed.

## Important Notes

1. **Time-series validation**: Never use future data to predict past races. Use `TimeSeriesSplit` for cross-validation.
2. **Data leakage**: Be careful not to include post-race information (actual odds at race time vs final odds).
3. **Horse racing specifics**:
   - Lane 1 advantage exists in boat racing, but not as strong in horse racing
   - Inside post positions have slight advantage in short races
   - Weather and track condition heavily affect results
4. **JRA specifics**:
   - 10 racecourses: Sapporo, Hakodate, Fukushima, Niigata, Nakayama, Tokyo, Chukyo, Kyoto, Hanshin, Kokura
   - Race grades: G1, G2, G3, Listed, Open, Conditions, Maiden

## Differences from Boat Racing

| Aspect | Boat Racing | Horse Racing |
|--------|-------------|--------------|
| Participants | 6 boats (fixed) | 8-18 horses (variable) |
| Post advantage | Lane 1 ~50% win rate | Slight inside advantage |
| Key factors | Motor, start timing | Blood, jockey, training |
| Data availability | Official free download | Paid (JRA-VAN) or scraping |
| Race frequency | Daily, 24 venues | Weekend mainly, 10 venues |

## Implemented Features

- ✅ All 5 bet types (Exacta, Trifecta, Quinella, Trio, Wide)
- ✅ Probability calibration (temperature scaling, binning)
- ✅ Walk-forward backtesting (Python + Rust CLI)
- ✅ Kelly criterion bet sizing
- ✅ Expected value filtering
- ✅ Live race scraper (netkeiba.com) - **Full Rust implementation**
- ✅ Single binary CLI (`live` command - no Python dependency)
- ✅ File-based cache with TTL (7 days for profiles, 24h for race card)
- ✅ Calibration in Rust CLI (`live`, `predict`, `serve` commands)
- ✅ Model retraining pipeline (`scripts/retrain.py`)
- ✅ Multiple model types (LightGBM, CatBoost, XGBoost, Ensemble)
- ✅ Hyperparameter optimization with Optuna (`--optimize` flag)
- ✅ Colored CLI output with progress bars
- ✅ Historical data scraper (db.netkeiba.com, 2022-2025)
- ✅ SQLite storage for historical race data
- ✅ Forward paper-trading loop (`paper-record` / `paper-settle` / `paper-report`) with
  official-payout settlement for verified ROI (see `docs/PAPER_TRADING.md`)
- ✅ Comprehensive test suite (213 Python + 109 Rust tests)

## Known Issues & Limitations

### Fixed: Feature Count Mismatch

~~The JSON API (`predict`, `backtest`) uses a 23-feature `HorseFeatures` struct (`types.rs`), but the ONNX model expects 39 features.~~

**Status**: ✅ Fixed. All commands now use unified 39-feature struct with `#[serde(default)]` for backward compatibility.

### Data Limitation: Post-Race Odds in Backtest

The Kaggle dataset only contains winning combination odds (post-race), not pre-race odds for all combinations. This makes backtest hit rates appear higher than real-world performance.

**Status**: ✅ Documented. Backtest CLI now displays warning:
```
WARNING: Kaggle dataset contains only winning combination odds (post-race).
         Hit rates and ROI may be overly optimistic. Use JRA-VAN for accurate results.
```

**Mitigation**: The reported +19.3% ROI should be considered optimistic. Real-world ROI will likely be lower. For accurate backtesting, integrate JRA-VAN data.

### Data Limitation: No Real Combination Odds Anywhere (ROI Unverified)

**The EV strategy is not yet validated on real combination odds.** Both data sources
lack pre-race odds for losing combinations, which the EV filter needs to decide bets:

- **Kaggle**: only winning-combination payouts (post-race).
- **netkeiba historical**: only final win odds (単勝) per horse + winning payouts. The
  full-combination odds page (`/odds/.../umatan/`) returns 404, and live odds are not
  retained after a race.

As a result, `backtest-historical` **estimates** combination odds from win odds (geometric
mean) when real odds are absent. It reads real odds from the `odds_snapshots` table when
present and reports the real-vs-estimated split, but no current scraper populates that
table. **Treat any ROI from estimated odds as indicative only, not verified.**

**Paths to real verification**: (1) forward paper-trading — capture live odds (the `live`
command already fetches them via netkeiba's JSON API) before post time, log EV bets, settle
after results; or (2) JRA-VAN historical odds (paid).

**Status**: ✅ Path (1) is now implemented as the `paper-record` → `paper-settle` →
`paper-report` command loop. It captures real pre-race combination odds, settles against the
**official payout** (haraimodoshi, parsed from the result page into `race_payouts`), and
reports verified ROI / hit-rate. No real races have been recorded yet, so no verified ROI
number exists *yet* — but the infrastructure to produce one now does. See
`docs/PAPER_TRADING.md`. (Captured odds also populate `odds_snapshots`, which
`backtest-historical` reads — improving historical backtest accuracy as a side effect.)

### Fixed: Live Betting Pipeline Never Produced Correct Bets

**Status**: ✅ Fixed (alongside paper-trading).

Building paper-trading surfaced two latent bugs in the `live` EV→odds→Kelly path that meant
it had never produced a correct recommended bet:

- **Combination keys never matched.** Predictions were keyed by netkeiba `horse_id` while
  odds were keyed by zero-padded post position, so the EV join always failed and `live`
  always printed "No bets meet the EV threshold". EV combos are now translated
  `horse_id` → post position before the odds lookup (`calculate_ev_exacta/trifecta` take an
  `id_to_post` map).
- **EV and Kelly disagreed on the odds unit.** Odds are decimal (e.g. `12.5`); EV used them
  directly (correct) but Kelly sizing divided by 100, collapsing every stake to the minimum
  unit. Kelly now uses decimal odds consistently via the `full_kelly_fraction` /
  `kelly_stake` helpers (shared by `live` and `paper-record`).

### Performance: Live Command Latency

~~Current `live` command takes ~60 seconds for 18 horses due to sequential profile fetching.~~

**Status**: Improved to ~30-40 seconds with parallel fetching (4 concurrent). Target <30s still in progress.

**See**: `docs/TODO_PERFORMANCE.md` for further optimization plan.

### Blood Features: Infrastructure Ready

**Status**: ✅ Infrastructure implemented. Sire/broodmare features are now computed during live prediction.

**Implemented**:
- `scripts/generate_sire_stats.py` - Generates sire statistics from cache
- `src/api/src/scraper/sire_stats.rs` - Rust sire stats loader
- 4 blood features: `sire_win_rate`, `sire_place_rate`, `broodmare_sire_win_rate`, `broodmare_sire_place_rate`

**Note**: Model currently uses 39 features (blood features excluded). To enable blood features, retrain model with JRA-VAN data and use `to_array_with_blood()` (43 features).

## Improvement Roadmap

See `docs/TODO_IMPROVEMENTS.md` for detailed improvement plans covering:
- UI/UX enhancements (input validation, error messages, output formatting)
- Model quality (blood features, feature importance, ensemble optimization)
- Performance (parallel fetching, connection pooling)
- Testing (scraper tests, integration tests, CI/CD)

## Future Extensions

- NAR (Regional racing) support
- JRA-VAN integration for real-time predictions with pre-race odds
- Production deployment (Docker, monitoring)
- Blood features (sire/broodmare aptitude)
- Real-time odds edge detection

## References

- Kaggle Dataset: https://www.kaggle.com/datasets/takamotoki/jra-horse-racing-dataset
- JRA Official: https://www.jra.go.jp/
- JRA-VAN (Paid): https://jra-van.jp/

---
> Source: [tsukasaI/keiba-ai](https://github.com/tsukasaI/keiba-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
