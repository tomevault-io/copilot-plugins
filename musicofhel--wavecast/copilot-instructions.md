## wavecast

> > **Status: ARCHIVED** — 66 experiments confirmed ~64% accuracy ceiling. No tradeable edge beyond the A2i production system (cron still runs). Research phase ended Feb 2026.

# WaveCast

> **Status: ARCHIVED** — 66 experiments confirmed ~64% accuracy ceiling. No tradeable edge beyond the A2i production system (cron still runs). Research phase ended Feb 2026.

Wavelet-shapelet financial forecasting library with SAX tokenization and transformer-based sequence prediction.

## Quick Start

```bash
cd ~/wavecast
source .venv/bin/activate
pytest tests/ -q          # 424 tests, ~9s on GPU
ruff check src/ tests/    # 0 errors
maturin develop --release # build Rust extension (optional, Python fallback available)
```

## Environment

- Python 3.12.12, venv at `.venv/`
- PyTorch 2.10.0+cu128, CUDA 13.1
- GPU: NVIDIA RTX 2060 SUPER (8GB VRAM) — auto-detected via `torch.cuda.is_available()`
- Rust 1.93.0 + maturin (build system for Rust/PyO3 extension)
- Data API: Massive.com — `MASSIVE_API_KEY` in `.env` (gitignored)
- `.env.example` has the template

## Project Layout

```
src/wavecast/
  core/         — types, config, exceptions, universe (DEFAULT=Phase3, LEGACY=Phase1-2)
  _rust.py      — Rust/PyO3 fallback wrapper (HAS_RUST flag)
  data/         — Massive.com fetcher, cache, preprocessing, storage, decomposition cache, MMapSequenceDataset
  wavelets/     — DWT decompose, CWT scalogram, reconstruction
  shapelets/    — W-TSS discovery, quality metrics, library, clustering
  dtw/          — DTW matching, ShapeDTW, similarity, subsequence search
  fractal/      — Hurst exponent, MFDFA, regime detection, self-similarity, HurstCache
  features/     — pipeline combining wavelet+shapelet+fractal+market+SAX features
  sax/          — PAA, SAX transform, Bag-of-Words + TF-IDF, price reconstruction
  tokenizer/    — SAXVocabulary, WaveletSAXTokenizer, sequence dataset builder
  models/       — WaveletLSTM, WaveletGPT (AMP), XGBoost, ensemble, registry, BatchPredictor
  evaluation/   — metrics (16 risk metrics), walk-forward backtest, token prediction eval, signal reporting
  signals/      — SignalGenerator, SignalBacktest, PositionSizer, TransactionCostModel, signal types & config
  experiments/  — ExperimentConfig, ExperimentResult, Runner, Splitter, Metrics, Storage, HPO
  forward/      — ForwardTestRunner, ForwardTestTracker, ForwardPrediction, report, config (paper trading)
  pipeline/     — stage orchestration, runner, library builder, token pipeline
  cli/          — Typer CLI: data, discover, match, analyze, forecast, library, backtest, sax, tokenize, experiment, signal, forward
  viz/          — scalogram, shapelet gallery, DTW alignment, forecast, fractal plots
rust/           — PyO3 extension crate (sax_words, bow, vocab, dataset acceleration)
tests/
  unit/         — 62 test files, 417 unit tests
  integration/  — 4 integration tests (pipeline, signal pipeline, performance, forward pipeline)
  fixtures/     — deterministic generators (seed=42)
scripts/        — experiment runners (run_C1-C7, run_D1_D4, run_F1/F4/F5/F6), fetch_phase3_universe.py, train_forward_model.py, model_audit.py
```

~115 source files, 69 test files, 424 tests passing.

## Architecture: Two Pipelines

### Pipeline 1: Wavelet-Shapelet Forecasting (Phase 1)
```
data → decompose → discover → match → fractal → features → forecast → evaluate
```
- Fetch OHLCV → DWT (db4, 5 levels) → shapelet discovery (W-TSS) → DTW matching
- Fractal analysis (Hurst, MFDFA) → ~40-50 feature vector
- Models: WaveletLSTM + XGBoost + weighted ensemble
- Evaluate: RMSE, MAE, directional accuracy, Sharpe, walk-forward backtest

### Pipeline 2: SAX Token Prediction (Phase 2) — **Primary pipeline**
```
data → decompose → SAX → tokenize → train WaveletGPT → evaluate
```
- DWT coefficients → PAA → SAX symbols → sliding window words → vocabulary
- Bag-of-Words + TF-IDF for cross-asset similarity
- WaveletGPT: causal transformer (token+position+level+sector embeddings)
- Weight tying between token embedding and output head (h=1); independent heads for h=2,4,8
- Multi-horizon prediction: horizons=[1,2,4,8], h=1-2 useful, h=4+ plateaus
- Evaluate: token accuracy, top-3 accuracy, directional accuracy (per-horizon)
- Phase 3 proved P2 dominates P1 — no ensemble benefit

### Pipeline 3: Signal Generation & Backtesting (Phase 5)
```
WaveletGPT.predict_proba() → SignalGenerator → PositionSizer → SignalBacktest → SignalBacktestResult
```
- Softmax probabilities aggregated by quartile bucket → P(up), P(down), P(flat); argmax = direction
- Temperature scaling calibration via NLL minimization on validation data
- Position sizing: fixed, linear (confidence × max), Kelly, fractional Kelly (0.5× Kelly with rolling lookback)
- Transaction costs: commission + spread (bps) + slippage (bps); direction changes double costs
- 16 risk metrics: Sharpe, Sortino, Calmar, max drawdown, profit factor, VaR, CVaR, win rate, avg win/loss ratio, expectancy, tail ratio, total/annualized return, volatility, num trades, avg trade return
- Per-trade records with entry/exit timestamps, gross/net returns, cost breakdown

### Production Trading System (A2i)
```
WaveletGPT.predict_proba() → compute_signal_b() → magnitude_filter(top_tercile) → flat_sizing → trade
```
- Model: d1_augmented_v1 (trained 2021-2025, embed_dim=64, 3 layers, context_length=16)
- Trade filter: top tercile by predicted magnitude (Signal B = expected absolute return from softmax)
- 2025 Test: 12,451 trades, 63.2% accuracy, Sharpe +8.40, expectancy +0.193%
- 2026 OOS: 1,644 trades, 66.7% accuracy, Sharpe +10.40, expectancy +0.339%
- Reversal filter tested and REJECTED (78.9% on 2025 collapsed to 63.8% on 2026)
- 66 experiments across 5 rounds confirmed ~64% econ_dir ceiling — no further model research warranted
- Full spec: `docs/PRODUCTION_SYSTEM.md`, results: `PHASE12_RESULTS.md`

### Pipeline 4: Forward Testing (Phase 7)
```
fetch_latest_bars → resolve pending → DWT/SAX/tokenize → WaveletGPT.predict_proba() → SignalGenerator → log prediction
```
- Paper trading on live/recent data — no broker integration
- `ForwardTestRunner.run_once()`: fetches latest bars, resolves prior predictions against actual prices, generates new predictions
- Reuses exact pipeline from ExperimentRunner (DWT → SAX → vocabulary encode → sequence dataset)
- Uses pre-trained model + vocabulary from disk (no retraining)
- `ForwardTestTracker`: JSONL-based prediction logging, resolution, rolling metrics (accuracy, directional accuracy, PnL, max drawdown)
- Per-ticker and per-interval metric breakdowns in `ForwardTestSummary`
- Text + JSON report generation via `generate_forward_report()` / `export_forward_json()`
- Storage: `~/.wavecast/forward_tests/{test_name}/predictions.jsonl`
- `fetch_latest_bars()`: fresh API fetch, 5x buffer for intraday (trading hours), 1.5x for daily+, no Parquet cache

### Performance Optimization (Phase 6)
- **Rust/PyO3 acceleration**: `extract_words`, `build_bow`, `build_corpus_tfidf`, `encode_batch`, `build_sliding_windows` — compiled Rust with Python fallback (`HAS_RUST` flag)
- **AMP mixed-precision**: `torch.autocast` + `GradScaler` in WaveletGPT fit/predict — no-op on CPU
- **Memory-mapped datasets**: `MMapSequenceDataset` — zero-copy `.npy` loading via `np.load(mmap_mode='r')`
- **Batch inference**: `BatchPredictor` with `predict_stream()`/`predict_proba_stream()` iterators, `torch.inference_mode()`
- Build: `maturin develop --release` (Rust 1.93 + maturin required)

### Experiment Framework (Phase 3)
```
ExperimentConfig → ExperimentRunner → walk-forward split → train → evaluate → ExperimentResult
```
- Walk-forward splitter: splits at RAW PRICE level before any DWT/SAX transformation
- ExperimentRunner: full pipeline per-split with bootstrap 95% CIs
- Three baselines: most-frequent-token, persistence, momentum
- Level-0 directional accuracy for multi-level SAX (detail levels use token accuracy only)
- Results stored as JSON in `~/.wavecast/experiments/`

## Key APIs

```python
# Phase 1
from wavecast.data.sources import fetch_massive
from wavecast.wavelets.dwt import decompose
from wavecast.shapelets.discovery import discover_shapelets
from wavecast.dtw.matching import match_against_library
from wavecast.fractal.hurst import wavelet_hurst
from wavecast.features.pipeline import FeaturePipeline
from wavecast.models.ensemble import EnsembleModel
from wavecast.pipeline.runner import PipelineRunner

# Phase 2
from wavecast.sax.sax import sax_transform, sax_distance
from wavecast.sax.bow import extract_words, build_bow, build_corpus_tfidf
from wavecast.tokenizer.vocabulary import SAXVocabulary
from wavecast.tokenizer.tokenizer import WaveletSAXTokenizer
from wavecast.tokenizer.dataset import build_sequence_dataset
from wavecast.models.wavelet_gpt import WaveletGPT
from wavecast.evaluation.token_eval import evaluate_token_predictions
from wavecast.core.universe import get_universe, DEFAULT_UNIVERSE, LEGACY_UNIVERSE, PHASE3_UNIVERSE
from wavecast.pipeline.token_pipeline import TokenPipelineRunner

# Phase 3 Experiments
from wavecast.experiments.config import ExperimentConfig
from wavecast.experiments.result import ExperimentResult
from wavecast.experiments.runner import ExperimentRunner
from wavecast.experiments.splitter import walk_forward_split
from wavecast.experiments.metrics import level0_directional_accuracy, compute_bootstrap_ci, compute_baselines
from wavecast.experiments.storage import save_results, load_results, compare_results
from wavecast.experiments.splitter import expanding_window_split, rolling_window_split
from wavecast.experiments.hpo import run_sax_hpo, run_architecture_hpo
from wavecast.data.decomposition_cache import DecompositionCache
from wavecast.fractal.hurst import rolling_hurst_with_regimes, HurstCache

# Phase 4
from wavecast.sax.reconstruction import reconstruct_price_delta, tokens_to_direction, PriceReconstructionResult

# Phase 5 Signals
from wavecast.signals import SignalGenerator, SignalBacktest, PositionSizer, TransactionCostModel
from wavecast.signals import TradingSignal, SignalSeries, TradeRecord, SignalBacktestResult
from wavecast.signals.config import SignalConfig, PositionSizingConfig, TransactionCostConfig, SignalBacktestConfig
from wavecast.evaluation.metrics import sortino_ratio, calmar_ratio, value_at_risk, conditional_var
from wavecast.evaluation.metrics import win_rate, avg_win_loss_ratio, expectancy, tail_ratio
from wavecast.evaluation.reporting import generate_signal_report

# Phase 6 Performance
from wavecast._rust import HAS_RUST  # True if Rust extension compiled
from wavecast.data.mmap_dataset import MMapSequenceDataset
from wavecast.models.batch_inference import BatchPredictor

# Phase 7 Forward Testing
from wavecast.forward import ForwardTestRunner, ForwardTestTracker, ForwardTestConfig
from wavecast.forward import ForwardPrediction, ForwardTestSummary
from wavecast.forward import generate_forward_report, export_forward_json
from wavecast.data.sources import fetch_latest_bars
```

## CLI

```bash
# Data
wavecast data fetch AAPL --start 2020-01-01 --interval 1d

# Phase 1
wavecast discover run AAPL
wavecast match find AAPL --against SPY
wavecast analyze hurst AAPL
wavecast forecast run AAPL --model ensemble
wavecast library build --universe default
wavecast backtest run AAPL

# Phase 2
wavecast sax transform AAPL --segments 512 --alphabet 7
wavecast sax bow AAPL --word-length 4
wavecast tokenize vocab --universe default
wavecast tokenize run --universe default --epochs 80

# Phase 3 Experiments
wavecast experiment run --tickers AAPL,MSFT --train-end 2023-12-31 --test-start 2024-01-01
wavecast experiment sweep --param alphabet_size --values 3,5,7,9,11
wavecast experiment show C1_granularity
wavecast experiment compare C1_granularity C2_level_contribution

# Phase 4
wavecast sax reconstruct AAPL              # price reconstruction from SAX tokens

# Phase 5 Signals
wavecast signal generate MODEL_PATH TICKER --horizon 1 --confidence-threshold 0.5
wavecast signal backtest MODEL_PATH TICKERS --train-end 2023-12-31 --test-start 2024-01-01 --position-method fractional_kelly
wavecast signal calibrate MODEL_PATH TICKERS --validation-start 2024-01-01 --validation-end 2024-06-30

# Phase 7 Forward Testing
wavecast forward run --model-path ~/.wavecast/models/forward_ready --vocab-path ~/.wavecast/models/forward_ready/vocabulary.json --tickers AAPL,MSFT,GOOGL,SPY --interval 1h --test-name paper_v1
wavecast forward status --test-name paper_v1
wavecast forward report --test-name paper_v1 --format json --output report.json
wavecast forward list
```

## Config System

Pydantic `BaseSettings` hierarchy in `core/config.py`:
- `WaveletConfig` — wavelet family, decomposition level, mode
- `ShapeletConfig` — z-threshold, min_length, top_k, IG minimum
- `DTWConfig` — window, pruning, normalization
- `FractalConfig` — Hurst method/window, MFDFA q-range, regime thresholds
- `ModelConfig` — LSTM/XGBoost hyperparams, ensemble weights
- `BacktestConfig` — capital, position size, commission, walk-forward splits
- `SAXConfig` — n_segments=512, alphabet_size=7, word_length=4, word_stride=1 (Phase 4 Optuna)
- `TokenizerConfig` — context_length=16, min_word_freq=1, max_vocab_size=100
- `SequenceModelConfig` — embed_dim=128, num_heads=4, num_layers=6, dropout=0.2, epochs=80, lr=0.0005, patience=15, use_amp=False, mmap_dataset_dir=None (Phase 4 Optuna + Phase 6 perf)
- `WaveCastConfig` — top-level aggregator with data/library/model/cache dirs + signal_backtest
- `SignalConfig` — confidence_threshold, calibration_method, temperature
- `PositionSizingConfig` — method (fixed/linear/kelly/fractional_kelly), max_position, kelly_fraction, min_position
- `TransactionCostConfig` — commission_rate, spread_bps, slippage_bps
- `SignalBacktestConfig` — initial_capital + nested signal/position_sizing/costs configs
- `ExperimentConfig` — full experiment specification (tickers, interval, SAX/model params, split dates)
- `ExperimentResult` — metrics + CIs + baselines + per-asset/sector/level breakdowns
- `ForwardTestConfig` — test_name, model_path, vocab_path, tickers, intervals, horizons, lookback_bars, dwt_levels, sax, signal, log_dir (standalone, not in WaveCastConfig)

## Type System

Core dataclasses in `core/types.py`:
- `TimeSeries`, `WaveletDecomposition`, `LevelStats`
- `Shapelet`, `ShapeletMatch`, `MatchResult`
- `HurstResult`, `MFDFAResult`, `SelfSimilarityResult`, `RegimeDetection`
- `FeatureVector` (wavelet + shapelet + fractal + market + sax features)
- `SAXRepresentation`, `SAXWord`, `TokenSequence`, `MultiLevelTokenSequence`
- Enums: `MarketLabel`, `RegimeType`, `AssetClass`, `Sector`

Signal dataclasses in `signals/types.py`:
- `TradingSignal` (timestamp, direction +1/-1/0, confidence, raw_probability, token_id, horizon, ticker)
- `SignalSeries` (list[TradingSignal]; properties: directions, confidences, timestamps)
- `TradeRecord` (entry/exit timestamps, direction, position_size, gross/net return, cost breakdown, confidence)
- `SignalBacktestResult` (returns, positions, equity_curve, timestamps, trades, metrics dict, config dict)

Forward dataclasses in `forward/types.py`:
- `ForwardPrediction` (id, timestamp, target_timestamp, ticker, interval, horizon, predicted_direction, predicted_confidence, predicted_token; resolution fields: actual_return, actual_direction, correct, resolved_at)
- `ForwardTestSummary` (test_name, start_time, tickers, intervals, prediction counts, accuracy, directional_accuracy, cumulative_pnl, max_drawdown, win_rate, per_ticker, per_interval)

## Exception Hierarchy

```
WaveCastError
├── DataError → DataNotFoundError
├── DecompositionError
├── ShapeletError → ShapeletLibraryError
├── DTWError
├── FractalError
├── ModelError → ModelNotTrainedError, SequenceModelError
├── PipelineError
├── SAXError
├── TokenizerError
└── ConfigError
```

## Model Audit Results (Phase 7, 2026-02-16) — CRITICAL

**The model has NO tradeable edge.** Run `scripts/model_audit.py` for full 13-analysis audit.

- **Economic directional accuracy: ~46%** (below 50% random). The reported 95.8% directional accuracy is SYMBOLIC — `level0_directional_accuracy` in `experiments/metrics.py` compares SAX word ordinals, not actual price direction.
- **Token accuracy (62.4%) is real but economically meaningless.** The model accurately predicts SAX tokens, but SAX tokens encode wavelet coefficient levels, not price direction.
- **Persistence is the dominant pattern**: 45.2% of predictions are "same as last token" with 79.2% accuracy. Change predictions are 48.5% (random).
- **Level 5's 87.4% token accuracy is persistence**: tokens change only 17.4% of the time. Mean run length = 7 tokens.
- **Fixed-sizing Sharpe: -0.5573**. Kelly's +0.18 is an artifact (near-zero position sizes).
- **Confidence is useless for trading**: token accuracy scales perfectly with softmax confidence (36%→93%), but economic directional accuracy is FLAT (~46%) across all deciles.
- **Model doesn't beat random**: real Sharpe at 84th percentile of random baseline (doesn't exceed P95).

**Implication**: Do NOT optimize the current pipeline further. The SAX representation is the bottleneck. Phase 8 must redesign the target variable (e.g., predict signed returns, return quantiles, or coefficient deltas instead of SAX levels).

Full results: `~/.wavecast/audit/model_audit_results.json`
Handoff: `.claude/handoff/2026-02-16-model-audit-results.md`

## Known Quirks

- `label_returns()` takes price series, computes returns internally — don't pass pre-computed returns
- `ShapeletLibrary.get()` raises on missing ID (doesn't return None)
- `ParquetCache.clear()` clears ALL cached data (no per-ticker clear)
- `FeaturePipeline()` takes no constructor args
- Hurst exponent can exceed 1.0 on integrated processes — mathematically correct, not a bug
- `shape_descriptor()` returns len-1 array (np.diff, no padding)
- `subsequence_search()` raises DTWError when query > series length
- `AssetClass.value` returns strings ('equity', etc.) not ints — use a mapping dict for numeric IDs. `Sector.value` same ('tech', 'finance', etc.)
- `SAXVocabulary` supports `len()` and `.size` property — both return total including PAD+UNK
- `build_sequence_dataset()` expects `MultiLevelTokenSequence` objects, not raw dicts. Accepts `max_horizon` param for multi-step prediction.
- WaveletGPT X format: `[context_token_0, ..., context_token_{L-1}, level_id, asset_class_id]`
- Weight tying means WaveletGPTNet vocab_size affects both embedding and output head simultaneously
- Cached OHLCV parquets use `{ticker}_1h_ohlcv.parquet` format (6 cols) — ExperimentRunner handles loading
- Rolling Hurst on raw prices always returns H > 1.0 (integrated processes) — use `use_returns=True` for regime detection on log returns
- `DEFAULT_UNIVERSE` (= `PHASE3_UNIVERSE`) has 20 US assets with sector tags; `LEGACY_UNIVERSE` has the original 19 (incl. crypto/forex). `get_universe("legacy")` for old universe.
- `AssetSpec` has optional `sector: Sector | None` field — set for DEFAULT_UNIVERSE, None for LEGACY_UNIVERSE
- Directional accuracy is level-0 only for multi-level SAX — detail levels represent oscillation magnitude, not price direction
- `SignalGenerator` uses same quartile boundaries as `token_eval.py`: midpoint=vocab_size//2, quarter=vocab_size//4. UP: tokens >= midpoint+quarter, DOWN: tokens <= midpoint-quarter
- `PositionSizer` Kelly methods fall back to linear sizing with fewer than 10 trade records in history
- `PositionSizer.min_position` uses strict `<` comparison — confidence exactly equal to min_position is NOT filtered out
- `TransactionCostModel.zero()` class method creates zero-cost model for testing
- `SignalBacktest` equity_curve has length n (not n+1) — initial capital is the first element scaled by first period return
- `HAS_RUST` is checked at import time — if Rust extension not compiled, all functions silently fall back to Python
- `maturin develop --release` must be re-run after any Rust source changes
- `BatchPredictor` accesses `_model._net`, `_model._device`, `_model._parse_x()` — tightly coupled to WaveletGPT internals
- `torch.autocast` with `enabled=False` is a complete no-op — safe to always wrap
- `MMapSequenceDataset` uses `mmap_mode='r'` (read-only) — cannot accidentally modify dataset files
- AMP only activates on CUDA devices; on CPU, `use_amp=True` is silently ignored
- `ForwardTestConfig` is standalone (not nested in `WaveCastConfig`) to avoid circular imports — `forward.config` imports `SAXConfig` from `core.config`
- `ForwardTestTracker` rewrites the full JSONL on resolution — file is small (hundreds of records), not a performance concern
- `ForwardTestTracker.resolve_pending()` finds the closest price at/before prediction time and at/after target time — tolerant of timestamp misalignment
- `ForwardTestRunner._build_pipeline_context()` reuses the exact DWT→SAX→tokenize pipeline from `ExperimentRunner._run_on_split()` but with pre-loaded vocabulary (no vocab rebuilding)
- `ForwardTestRunner` caches model + vocab on first `run_once()` call — subsequent calls within same process reuse them
- `ForwardTestRunner` accesses `model._config["context_length"]` to match the trained model's context length
- `fetch_latest_bars()` always fetches fresh (no cache), uses 5x buffer for intraday intervals (only ~7 trading hours/day, 5 days/week), 1.5x for daily+
- DWT level 5 with db4 requires minimum ~224 data points — default lookback_bars is 300 to ensure enough data after gap filtering
- Forward test CLI `report` command uses `--format` flag but `fmt` parameter name internally to avoid shadowing Python builtin

## Testing

```bash
pytest tests/ -q                                          # full suite, 424 tests (~9s GPU)
pytest tests/unit/ -q                                     # unit only (~5s)
pytest tests/unit/test_wavelet_gpt.py -v                  # GPU model tests (incl. AMP)
pytest tests/unit/test_experiment_runner.py -v             # experiment framework tests
pytest tests/unit/test_signal_backtest.py tests/unit/test_signal_generator.py -v  # signal tests
pytest tests/integration/test_signal_pipeline.py -v       # signal pipeline integration
pytest tests/unit/test_rust_acceleration.py -v             # Rust vs Python equivalence
pytest tests/unit/test_batch_inference.py -v               # batch inference tests
pytest tests/unit/test_mmap_dataset.py -v                  # memory-mapped dataset tests
pytest tests/integration/test_performance.py -v            # performance benchmarks
pytest tests/unit/test_forward_tracker.py tests/unit/test_forward_runner.py -v  # forward testing unit
pytest tests/integration/test_forward_pipeline.py -v      # forward testing integration
pytest tests/ --cov=wavecast --cov-report=term-missing    # coverage
ruff check src/ tests/                                    # lint
maturin develop --release                                 # rebuild Rust extension
```

## Benchmarks (RTX 2060 SUPER)

- Full test suite: 424 tests in 9s (was 2m20s CPU-only before GPU)
- WaveletGPT tests: ~5s (incl. AMP and mmap tests)
- WaveletGPT training (161K-600K params, 80 epochs): 4-10s depending on architecture
- SAX+BoW+TF-IDF on 7 assets x 2000 points: <1s (faster with Rust extension)
- Rust `extract_words`: 100 iterations on 10K chars in <1s
- Full 20-ticker experiment run (hourly, all levels): ~3-4 min
- Phase 3 full experiment suite (78 runs): ~2.5 hours

## Phase 3 Research Results (real data, 20 assets, hourly bars)

**Phase 3 config**: alphabet=7, levels=[1,2,5], context=16, vocab=100, min_freq=1, cross-sector training
**Phase 4 Optuna-optimized**: n_segments=512, embed_dim=128, num_layers=6, dropout=0.2, horizons=[1,2]

| Question | Answer |
|----------|--------|
| Q1: Alphabet size | **7** — best directional accuracy, good granularity/learnability balance |
| Q2: DWT levels | **[1,2,5]** — levels 3&4 are noise (removing them improves accuracy) |
| Q3: Cross-sector | **Multi-sector helps** 4/6 sectors, biggest gain for finance (+3.8%) |
| Q4: Vocabulary | **100 max, min_freq=1** — natural vocab is only 83 tokens, fully saturated |
| Q5: Context length | **16** — sweet spot; 32 marginal gain, 64 hurts (fewer samples) |
| Q6: Regime | **Consistent** — 82.7% overall, +10.3% over persistence, mean-reverting slightly best |
| Q7: Ensemble | **No benefit** — P2 dominates P1, combining hurts performance |

### 2025 Held-Out Evaluation (Final Model)
- Trained on 2021-2024, tested on 2025 (truly unseen during all experiments)
- Token accuracy: 60.8% [59.96%, 61.68%], directional accuracy: 95.8% [95.26%, 96.30%]
- Level 5 (coarsest): 87.4% token accuracy; levels 1-2: ~55%
- Top sectors: broad ETFs (62.5%), commodity ETFs (62.3%), tech (61.6%)
- Persistence baseline: 36.0%, momentum baseline: 38.0% — model lift: +56-60 pp
- No overfitting detected: 2025 results consistent with 2024 validation
- **CAVEAT (Phase 7 audit)**: These metrics are real for TOKEN prediction but have ZERO economic value. See "Model Audit Results" section above.

### Phase 4 Results (HPO + Multi-Horizon)
- **Optuna SAX**: n_segments=512 (was 256) — higher resolution helps
- **Optuna architecture**: embed_dim=128, num_layers=6, dropout=0.2 — larger model preferred
- **Multi-horizon**: h=1 (68.5% token acc), h=2 (56.3%), h=4+ plateaus at ~41%
- **Per-sector fine-tuning**: hurts ALL 6 sectors — cross-sector training definitively confirmed
- **Expanding window**: results robust across multiple evaluation windows

## Data Source

- **Massive.com** (formerly Polygon.io) — `MASSIVE_API_KEY` env var
- **Plan**: US Stocks, Unlimited API Calls, 5yr history, Minute Aggregates
- Package: `massive>=2.0` on PyPI
- Auto-pagination via `client.list_aggs()`, timestamps in Unix ms
- Results cached to `~/.wavecast/cache/` as Parquet (40 files, 11.9 MB for Phase 3)
- **DEFAULT_UNIVERSE** (= PHASE3_UNIVERSE): 20 US assets across 6 sectors (tech, finance, energy, healthcare, broad ETFs, commodity ETFs)
- **LEGACY_UNIVERSE**: 19 assets across equity/crypto/forex/commodity (Phase 1-2)
- Hourly bars: ~4,875 per asset for training period (2021-2023)
- All commodity ETFs (GLD, SLV, USO, UNG) verified working on US Stocks plan

---
> Source: [musicofhel/wavecast](https://github.com/musicofhel/wavecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
