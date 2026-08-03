## quant-trade

> Hybrid architecture crypto quantitative trading system for personal use + research.

# Crypto Quant Trading System

Hybrid architecture crypto quantitative trading system for personal use + research.
Medium-low frequency, multi-coin (BTC + altcoins), spot + perpetuals.

**项目状态 & 待办：** `docs/superpowers/PROJECT_STATUS.md` — 每次 session 开始前读这个文件了解当前进度和下一步任务。

## Architecture (4-layer)

```
Layer 1: Data Hub → Layer 2: Factor Studio → Layer 2.5: Risk Gate → Layer 3: Trading Engine (freqtrade)
```

- **Data Hub:** Multi-source ingestion (CEX via ccxt, on-chain, sentiment, valuation). Thin adapters → unified schema.
- **Factor Studio:** Factor library (YAML), auto mining (gplearn/LightGBM), evaluation (IC/IR), signal generation. Human-in-the-loop — system suggests, human decides.
- **Risk Gate:** Independent process. Layer 0: structural safety (hardcoded, 2 rules). Layer 1: all configurable (YAML). Layer 2: anomaly detection (independently toggleable). Fail-closed on crash.
- **Trading Engine:** Freqtrade for backtesting + live trading. File-based signal bus (MVP — JSONL + .ready flag).

**Design doc:** `docs/superpowers/specs/2026-05-12-crypto-quant-trading-system-design.md`

## Key Principles

1. **Adapter pattern:** New data source = thin adapter (~50-400 LOC). Zero downstream changes.
2. **Signal bus decoupling:** Factor Studio → JSONL signal file → Risk Gate → freqtrade. Clean interface.
3. **Human-in-the-loop:** System suggests, human decides. No autonomous parameter changes.
4. **MVP-first:** Parquet+DuckDB only. File polling only. Date-partitioned files. Add complexity when proven necessary.
5. **All parameters configurable:** Only 2 structural safety rules are hardcoded. Everything else is YAML.
6. **Interface contracts:** All schemas in `contracts.py`. Tests as executable documentation.

## Directory Structure

```
quant/
├── data_hub/            # Layer 1 - collectors, pipeline (cleaner + validator + store), features
├── factor_studio/        # Layer 2 - library (YAML), mining, evaluation, signal
├── risk_gate/            # Layer 2.5 - independent process, fail-closed
├── trading/              # Layer 3 - freqtrade strategies, signal receiver (file poller)
├── adapters/             # Thin adapters for new data sources
├── contracts.py          # Global interface definitions
├── config/               # Per-mode + per-strategy YAML configs
├── data/                 # Date-partitioned Parquet files
├── tests/                # One test file per module
└── notebooks/            # Jupyter exploration
```

## Development Modes

| Mode | Storage | Command |
|------|---------|---------|
| Dev | Parquet + DuckDB | `make dev` |
| Research | Parquet + DuckDB | `make research` |
| Live | Parquet + DuckDB + file signal bus | `make live` |

## Signal Format (Interface Contract)

```json
{"signal_id":"uuid","timestamp":"...","symbol":"TOKEN/USDT","signal":"BUY/SELL",
 "strength":0.82,"position":1.0,"stop_loss":null,"take_profit":null,
 "strategy":"...","factors":{...},"metadata":{"timeframe":"15m","expires_at":"..."}}
```

- signal_id: required, UUID for full audit trail
- stop_loss/take_profit: optional, strategy-custom prices. Absent → Risk Gate default formula
- entry_price: intentionally NOT included. Execution price determined at trade time.

## Data Collection Priority (by real-trading ROI)

| Priority | Data | Source |
|----------|------|--------|
| P0 | OHLCV, funding rate, OI, exchange netflow | ccxt |
| P1 | Stablecoin mint/burn, TVL, listing announcements | DeFiLlama, CryptoPanic |
| P2 | Twitter mentions, partnership events, vesting unlock | Twitter API, public tokenomics |
| P3 | Whale tracking, multisig, Telegram/Discord, NLP | Various (low ROI) |

## Data Cleaning Rules

- Dedup: keep last for duplicate timestamps
- Missing: forward fill, max 5 consecutive candles
- Outlier: FLAG ONLY, NEVER DROP. Crypto extreme values are real signal.
- Validation: price>0, volume>=0, timestamp monotonic, OHLC consistent

## Session Context

- Each module built and tested independently. One module per session.
- Every module has a test file.
- After completing each module, update "Completed Modules" below.
- New session: Claude reads this file → knows architecture, progress, next step.
- Interface contracts in `contracts.py` define all data shapes.

## Completed Modules
**Phase 1 COMPLETE** — all 11 tasks done, 30 tests passing, E2E pipeline verified.

- Task 1: `quant/contracts.py` — all interface definitions
- Task 2: `quant/data_hub/collectors/market.py` — MarketCollector (ccxt OHLCV + derivatives)
- Task 3: `quant/data_hub/pipeline/cleaner.py` + `validator.py` — dedup, gap fill, outlier flag
- Task 4: `quant/data_hub/pipeline/store.py` — DuckDBStore (Parquet date-partitioned)
- Task 5: `quant/data_hub/features/preprocessing.py` — SMA, returns, volatility, price position
- Task 6: `quant/factor_studio/library/loader.py` + `pump_coin_factors.yaml` — 5 initial factors
- Task 7: `quant/factor_studio/signal/generator.py` + `evaluation/evaluator.py` — IC-weighted signal gen
- Task 8: `quant/risk_gate/gate.py` — Layer 2.5, fail-closed, YAML-configurable
- Task 9: `quant/trading/strategies/signal_receiver.py` — freqtrade strategy + JSONL signal bus
- Task 10: `quant/adapters/alpha_klines.py` — BN Alpha thin adapter
- Task 11: `Makefile` + `quant/tests/test_e2e_backtest.py` — E2E pipeline verified

**Phase 2 Core COMPLETE** — all 11 tasks done, 55 tests passing, freqtrade backtest verified.

- Task 1: `conda env freqtrade` (Python 3.11.15, freqtrade 2026.4) + `requirements-phase2.txt`
- Task 2: derivatives history pipeline — `fetch_funding_rate_history`, `fetch_open_interest_history`, `write/read_derivatives` in DuckDBStore
- Task 3: YAML-driven features — `config/features/default.yaml` + `config_loader.py`, `compute_features` config-driven
- Task 4: factor formula compiler — AST-guarded `FactorCompiler`, whitelist + sign翻转, `pump_coin_factors.yaml` f_oi_surge formula fixed
- Task 5: Evaluator IR — `compute_ir` across periods
- Task 6: Evaluator decay — `factor_studio/evaluation/decay.py`, IC by horizon
- Task 7: Evaluator stability — `factor_studio/evaluation/stability.py`, rolling IC stats
- Task 8: Evaluator multi-regime — `compute_multi_regime_ic` (bull/ranging/bear)
- Task 9: Walk-forward backtester — `factor_studio/backtest/walk_forward.py` + `replay_writer.py`, timestamped JSONL output
- Task 10: freqtrade integration — `SignalReceiver` inherits `IStrategy`, spot mode (`can_short=False`), backtest verified
- Task 11: CLAUDE.md + memory updated

**Phase 2 Extensions — Factor Warehouse COMPLETE** — 2026-05-14

- `master_factors.yaml` — central factor warehouse, 81 factors across 7 categories
- `param_grid` auto-expansion in FactorLoader — `{placeholder}` substitution, `itertools.product`
- 12 new feature kinds: kmid, klen, kup, klow, ema, macd, bollinger, wr, bias, adx (multi-output: macd/bollinger/adx)
- 40 new feature specs in `config/features/default.yaml` (66 total columns)
- Compiler extended: `clip`, `log` in `_SAFE_FUNCS`
- 8 validation tests in `test_master_factors.py`, all passing
- Plan: `docs/superpowers/plans/2026-05-14-alpha158-factor-warehouse.md`

**Key decisions made in Phase 2 Core:**
- IC weight: `abs(IC)` as weight + `sign(IC)` to flip factor direction (not raw IC)
- Walk-forward step = test_window (non-overlapping, statistically clean); YAML-configurable
- Signal files: timestamped `signals_YYYYMMDD_HHMMSS.jsonl` (not overwrite)
- freqtrade: spot mode only for now; futures needs `trading_mode: futures` + `can_short=True`
- Human-in-the-loop alerts: Feishu (Lark) webhook, NOT Telegram

## Next Module
Phase 2 Extensions — remaining items (each gets its own plan in `docs/superpowers/plans/`):
- ~~Factor Warehouse~~ — DONE (2026-05-14): master_factors.yaml, 81 factors, param_grid expansion
- Auto Mining Engine (gplearn + LightGBM candidate generator)
- Factor Manager Streamlit dashboard (base dashboard exists at `tools/dashboard.py`)
- Feishu (Lark) human-in-the-loop bot (user provides webhook URL)
- Paper trading live mode
- P1/P2 data sources (stablecoin, TVL, listings, Twitter, vesting)

---
> Source: [AbsoluteZhc/quant-trade](https://github.com/AbsoluteZhc/quant-trade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
