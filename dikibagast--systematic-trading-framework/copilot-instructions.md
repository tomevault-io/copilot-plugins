## systematic-trading-framework

> **Read this file FIRST.** It determines which path to follow based on your task.

# AGENTS.md — Agent Router & Development Guidelines

**Read this file FIRST.** It determines which path to follow based on your task.

---

## 🚦 Quick Decision Tree

```
What are you working on?
├─ Creating/modifying a TRADING STRATEGY?
│  └─ Go to Mode A → Read docs/STRATEGY.md → Scope: strategies/{name}/
└─ Modifying the FRAMEWORK itself?
   └─ Go to Mode B → Read docs/FRAMEWORK.md → Scope: engine/, data/, run.py, config/
```

---

## 🔧 Build, Test, and Run Commands

```bash
# Quick backtest (no optimization) — primary testing method
python run.py --strategy ema_crossover_rsi --mode backtest

# Backtest with detailed report
python run.py --strategy ema_crossover_rsi --mode backtest --save-reports

# Full validation (WFO + Monte Carlo) — SLOW (~hours)
python run.py --strategy ema_crossover_rsi --mode full

# Position sizing optimization (Kelly Criterion)
python run.py --strategy ema_crossover_rsi --mode optimize_sizing --apply

# Fetch OHLCV data
python data/fetcher.py --symbol BTC/USDT:USDT --timeframe 15m

# Inspect data quality
python data/inspect.py --symbol BTC/USDT:USDT --timeframe 15m
```

**⚠️ CRITICAL WARNING:** Never run `--mode full` during feature development. Use `--mode backtest` for quick feedback.

**🧪 Testing Strategy:** No formal test suite — use backtest mode: `python run.py --strategy {name} --mode backtest`

---

## 📐 Code Style Guidelines

### Import Organization (MANDATORY)

```python
# 1. Standard library
from pathlib import Path
from typing import Any, Dict, List, Optional, Tuple

# 2. Third-party
import polars as pl
import yaml

# 3. Local imports
from engine.base_strategy import BaseStrategy
```

### Type Hints (MANDATORY)

```python
def run_backtest(
    signals: pl.DataFrame,
    config: dict,
    position_sizing_mode: str = "percent_equity",
) -> Tuple[pl.DataFrame, pl.DataFrame]:
    """Run vectorized backtest on signal data."""
    pass
```

### Docstrings (Google Style)

```python
def calculate_sharpe(equity_curve: pl.DataFrame, risk_free_rate: float = 0.0) -> float:
    """Calculate annualized Sharpe Ratio.

    Args:
        equity_curve: DataFrame with 'equity' column (f64)
        risk_free_rate: Annual risk-free rate (default 0.0)

    Returns:
        Annualized Sharpe ratio as float

    Raises:
        ValueError: If equity curve has fewer than 2 data points
    """
    pass
```

### Naming Conventions

- Classes: `PascalCase`
- Functions/variables: `snake_case`
- Private functions: `_leading_underscore`
- Constants: `UPPER_SNAKE_CASE`

### Polars Usage (MANDATORY — NO PANDAS IN STRATEGY/ENGINE CODE)

```python
# ✅ CORRECT
df = df.with_columns([
    pl.col("close").ewm_mean(alpha=0.1).alias("ema")
])
signals = pl.when(condition).then(1).otherwise(-1)

# ❌ WRONG — NEVER USE PANDAS in strategy or engine code
import pandas as pd
```

**Exception**: The `visualization/` directory may use pandas for Streamlit/Plotly display purposes. All backtesting, strategy signal generation, and validation logic must use polars exclusively.

### Error Handling

```python
def run_backtest(signals: pl.DataFrame, config: dict) -> Tuple:
    required_cols = {"timestamp", "open", "high", "low", "close", "volume", "signal"}
    if not required_cols.issubset(set(signals.columns)):
        missing = required_cols - set(signals.columns)
        raise ValueError(f"Missing required columns: {missing}")
```

### Configuration Management

All strategy parameters MUST be in `config.yaml`. Access via `self.params.get("key", default)`.

---

## 🎯 Mode A: Creating a New Strategy

**When:** Creating/modifying strategies, signals.py, config.yaml, parameter spaces
**Scope:** `strategies/{name}/` directory only
**Docs:** Read `docs/STRATEGY.md` first

---

## 🔨 Mode B: Modifying the Framework

**When:** Changing backtesting, adding metrics, modifying data fetching
**Scope:** `engine/`, `data/`, `run.py`, `config/`
**Docs:** Read `docs/FRAMEWORK.md` first

---

## ⚠️ Hard Rules

- **NEVER** use pandas — use polars only
- **NEVER** use `as any`, `@ts-ignore`, `@ts-expect-error`
- **NEVER** install new dependencies without approval
- **NEVER** hardcode values — use YAML configs
- **NEVER** leave TODO comments — fully implement
- **NEVER** commit unless explicitly requested
- NO live trading code

---

## 📚 Documentation Index

| Document                  | Purpose                          |
| ------------------------- | -------------------------------- |
| `docs/STRATEGY.md`        | Strategy development guide       |
| `docs/FRAMEWORK.md`       | Framework architecture guide     |
| `docs/POSITION_SIZING.md` | Kelly Criterion optimization     |
| `README.md`               | Project overview and quick start |

---

## 🔄 Workflow

1. Start here (AGENTS.md) → Determine mode (A or B)
2. Read relevant guide (STRATEGY.md or FRAMEWORK.md)
3. Follow existing patterns in codebase
4. Use polars for all data operations
5. Add type hints to all functions
6. Run backtest to verify: `python run.py --strategy {name} --mode backtest`
7. Commit with clear message (only when requested)

**Remember:** Stay in your scope. Framework changes affect all strategies.

---
> Source: [dikibagast/systematic-trading-framework](https://github.com/dikibagast/systematic-trading-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
