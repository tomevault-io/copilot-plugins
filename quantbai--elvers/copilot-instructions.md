## elvers

> Polars-native quantitative research platform. This memo is for AI assistants working on the codebase.

# Elvers -- AI Development Memo

Polars-native quantitative research platform. This memo is for AI assistants working on the codebase.

For contribution workflow, numerical invariants, and coding standards, see [CONTRIBUTING.md](CONTRIBUTING.md).
For operator specifications, see [OPERATORS.md](OPERATORS.md).

---

## Conduct

**When uncertain, ask. Do not guess.** Flag assumptions explicitly: "I am assuming X. Is that correct?"

- **Design before implementing.** Think through module placement, API surface, and long-term implications before writing code.
- **Write cold, factual prose.** No emotional language. No marketing. State facts and trade-offs.
- **Describe upstream behavior, do not complain.** Say "Polars applies ddof asymmetrically; Elvers isolates this by using ddof=1" -- not "Polars has a bug".
- **`__init__.py` is for imports/exports only.** New functionality goes in dedicated modules.
- **Each module has one responsibility.** Do not let a module exceed its scope. If unsure where code belongs, ask.

---

## Architecture

```
elvers/
  __init__.py                Imports/exports only. __version__ is the single version source.
  _meta.py                   Diagnostics (show_versions)

  core/
    factor.py                Factor (column name + Panel ref, zero data storage)
    panel.py                 Panel (single DataFrame, _add_col + memoization)

  ops/                       Step 4: Factor computation (72 operators)
    base.py                  Arithmetic
    timeseries.py            Time-series (per-symbol rolling window)
    cross_sectional.py       Cross-sectional (across symbols per timestamp)
    math.py                  Math (element-wise)
    neutralization.py        Neutralization and group operators
    _dev.py                  Experimental (not exported)
    _validation.py           Input validation helpers

  data/                      Step 2+3: Data acquisition and storage
    providers/               Exchange adapters (binance, okx, local)
    store.py                 Parquet on disk (incremental update)
    loader.py                DataFrame -> Panel (schema validation + balance)
    sample/                  Built-in sample data

  universe/                  Step 1: Instrument selection and filtering
  analysis/                  Step 5: IC, decay, turnover, coverage, correlation
  synthesis/                 Step 6: Orthogonalization, combination, selection
  portfolio/                 Step 7: Optimization, constraints
  backtest/                  Step 8: Unified signal -> PnL engine
  risk/                      Step 9: Exposure, limits, VaR
  execution/                 Step 10+11: Trading + post-trade analysis
  monitor/                   Step 12: Dashboard, alerts, logging

tests/
  conftest.py                Fixtures: make_ts, make_factor, make_panel_ts, make_panel_cs
  test_*.py                  One file per module
```

## Core Patterns

- Time-series operators: `expr.over("symbol")`
- Cross-sectional operators: `expr.over("timestamp")`
- All operators: `Factor -> Factor`, stateless, functional
- Binary/multi-factor operators: all factors must share the same Panel (`_check_panel`)
- Factor.name = column name in Panel._df = human-readable expression
- `_add_col` skips computation if column already exists (memoization)

## Data Flow

```
data/providers -> data/store -> data/loader -> Panel
                                                 |
Panel["close"] -> Factor -> ops -> computed Factor
                                                 |
            analysis (IC, decay, turnover) -> report
                                                 |
      synthesis (orthogonalize -> combine) -> alpha Factor
                                                 |
               backtest (signal -> PnL) -> metrics
                                                 |
            execution (rebalance -> orders) -> trades
```

Backtest accepts any `signal()` output: single factor, multi-factor composite, or portfolio-optimized weights. Same interface, same output format.

---

## Operator Template

```python
def ts_example(f: Factor, window: int) -> Factor:
    name = f"ts_example({f.name},{window})"
    expr = pl.col(f._col).rolling_mean(window, min_samples=window).over("symbol")
    f.panel._add_col(expr, name)
    return Factor(name, f.panel)
```

After implementing:
1. Add to `ops/__init__.py` imports and `__all__`
2. Add tests in matching `test_*.py`
3. Add entry to `OPERATORS.md`

---

## Known Limitations

- `trade_when`: sentinel value (-1.79e308) should become struct-based
- `_validation.py`: not wired into all operator entry points
- `_dev.py`: Python callbacks, not production-grade
- No property-based testing or performance benchmarks
- Column name collisions possible if same operator is called with identical name but different semantics
- Steps 1-3, 5-12 are scaffolded but not yet implemented

---

## Next Steps (0.4.0 -> 0.5.0)

Current status: Step 4 (Factor Computation) is complete with 72 operators. Column-based architecture, memoization, pyright CI all in place. Tests need rewrite for new architecture.

Reference implementation: `ref/phandas_dev/` contains a pandas-based version with data providers, backtesting, analysis, and OKX execution. All modules below should be Polars-native rewrites, not pandas ports.

### Immediate (0.4.0 release blocklist)

1. **Rewrite tests for column-based architecture.** Current tests use old Factor(DataFrame) pattern. Every test must use Panel + Factor(column_name, panel). Oracle cross-validation against NumPy/SciPy for complex operators.

2. **Loader strictness.** `load()` should reject anything not matching the Panel contract. No hand-holding validation, no auto-conversion. Data cleaning is upstream's job. Accept: pl.DataFrame with (timestamp: Date|Datetime, symbol: Utf8, + numeric columns). Reject everything else with a clear error.

### Phase 1: Analysis (Step 5)

Module: `elvers/analysis/`

```
analysis/
  __init__.py
  ic.py              Information Coefficient (Spearman rank-corr vs forward returns)
  decay.py           IC decay across horizons (1d, 2d, 5d, 10d, 20d)
  turnover.py        Factor turnover (rank change between periods)
  coverage.py        Non-null ratio per timestamp
  correlation.py     Factor-to-factor correlation matrix
  report.py          Aggregated FactorReport object
```

Input: Factor + price Factor + horizons list.
Output: FactorReport with .ic(), .decay(), .turnover(), .coverage(), .correlation().
Reference: `ref/phandas_dev/phandas/phandas/analysis.py`

### Phase 2: Synthesis (Step 6)

Module: `elvers/synthesis/`

```
synthesis/
  __init__.py
  orthogonalize.py   Gram-Schmidt or regression residual decorrelation
  combine.py         Equal-weight, IC-weighted, optimized-weight combination
  select.py          Factor selection by IC threshold, turnover filter, coverage filter
```

Phandas only does arithmetic combination (`(f1 + f2 + f3) / 3`). A top fund adds:
- **Orthogonalization**: remove redundancy between factors before combining
- **IC-weighted**: weight by rolling IC (factors with higher predictive power get more weight)
- **Selection**: drop factors below IC threshold or with excessive turnover

### Phase 3: Backtest (Step 8)

Module: `elvers/backtest/`

```
backtest/
  __init__.py
  engine.py          Signal -> position weights -> daily PnL
  cost.py            Transaction cost models (fixed, linear, market impact)
  metrics.py         Sharpe, Sortino, Calmar, max drawdown, turnover, linearity
  report.py          BacktestReport with .summary(), .equity(), .drawdowns()
```

Phandas supports dollar-neutral only. Add:
- Long-only mode
- Custom weight functions (not just equal-weight signal)
- Multi-period holding (not just daily rebalance)
- Configurable cost models (fixed rate, linear impact, square-root impact)

Reference: `ref/phandas_dev/phandas/phandas/backtest.py`

### Phase 4: Risk (Step 9)

Module: `elvers/risk/`

```
risk/
  __init__.py
  exposure.py        Factor exposure decomposition (market, sector, style)
  limits.py          Position limits, concentration limits, turnover limits
  var.py             Value-at-Risk (historical, parametric)
```

### Phase 5: Execution (Step 10+11)

Module: `elvers/execution/`

```
execution/
  __init__.py
  rebalancer.py      Target weights -> order list (exchange-agnostic)
  adapters/
    okx.py           OKX perpetual swap adapter
    binance.py       Binance adapter
  twap.py            TWAP/VWAP execution algorithms
  post_trade.py      Slippage analysis, execution quality
```

Reference: `ref/phandas_dev/phandas/phandas/trader.py` and `ref/phandas_dev/jinglabs/`

Note: Execution adapters contain API credentials structure. The adapter interface is public; credential handling and proprietary execution logic are private.

### Phase 6: Data + Universe (Steps 1-3)

Module: `elvers/data/`, `elvers/universe/`

```
data/
  providers/
    base.py          Provider interface
    binance.py       Binance OHLCV
    okx.py           OKX perpetual swap data
    local.py         Local parquet/csv
  store.py           Incremental parquet storage
  loader.py          DataFrame -> Panel (strict validation + balance)
  sample/            Built-in sample data (crypto_1d.parquet)

universe/
  __init__.py
  filter.py          Liquidity, volume, listing date filters
  groups.py          Sector/category classification
```

Reference: `ref/phandas_dev/phandas/phandas/providers/` and `ref/phandas_dev/phandas/phandas/data.py`

### Phase 7: Monitoring (Step 12)

Module: `elvers/monitor/`

Deferred until execution layer is stable. Will include position monitoring, PnL tracking, and alerting.

### Open Questions (discuss with maintainer)

1. Execution layer: public or private? The adapter interface is safe to open-source. Proprietary TWAP/execution logic may stay private.
2. Synthesis depth: start with equal-weight + IC-weighted, or include PCA/ML from day one?
3. Backtest scope: dollar-neutral only (crypto standard), or support long-only and long-short?
4. Data providers: which exchanges to support first? Binance + OKX from phandas, or add more?

---
> Source: [quantbai/elvers](https://github.com/quantbai/elvers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
