## ai-quant-framework

> > This file is the entry point for AI agents. Keep it concise, let agents dig deeper as needed.

# AGENTS.md - AI Agent Navigation

> This file is the entry point for AI agents. Keep it concise, let agents dig deeper as needed.

---

## Project Overview

- **Project**: AI Quant Framework (ai_quant)
- **Type**: Event-driven quantitative trading backtesting framework with native AI integration
- **Python Version**: 3.11+ (see `.python-version`)
- **Package Manager**: uv / pip
- **PyPI Package**: `ai-quant-framework`
- **Highlights**: SQLite/CSV/T3Time data sources, rule + AI strategies, HTML reporting

## Quick Start

1. Install: `pip install -e src`（T3Time 可选：`pip install -e "src[t3time]"`）
2. 规则回测: `python -m ai_quant.main src/ai_quant/config.yaml`
3. T3Time 回测: `python -m ai_quant.main src/ai_quant/config_t3time.yaml`
4. Test: `pytest`

## Architecture

Event-driven architecture with 7 layers + reporting:

```
Data → Strategy → Risk → Execution → Portfolio → Analysis → Reporting
  ↓        ↓         ↓         ↓         ↓
BarEvent SignalEvent OrderEvent FillEvent
```

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Core Principles

1. **Event-Driven**: Components communicate via EventBus
2. **Plugin Architecture**: Inherit base classes; register in Registry
3. **Configuration-Driven**: All parameters via YAML
4. **Type Safety**: Full annotations, strict MyPy
5. **Mark-to-market equity**: Drawdown/Sharpe from Portfolio equity, not fill cashflows

## Code Standards

- **Formatting**: Ruff (`ruff format .`)
- **Type Check**: MyPy (`mypy src/`)
- **Single File Limit**: 350 lines
- **Docstrings**: Google Style

## Module Index

| Module | Path | Purpose |
|--------|------|---------|
| Models | `src/ai_quant/core/models.py` | Bar, Signal, Order, Fill, Trade, Direction, OrderType, Position |
| Events | `src/ai_quant/core/events.py` | EventBus, EventType, event factory functions |
| Exceptions | `src/ai_quant/core/exceptions.py` | Exception hierarchy |
| Data | `src/ai_quant/data/` | csv / sqlite / multistock_csv / t3time_csv |
| Strategy | `src/ai_quant/strategy/` | ma_cross / rsi / bollinger / t3time |
| Risk | `src/ai_quant/risk/` | SimpleRiskManager / FullRiskManager |
| Execution | `src/ai_quant/execution/` | BacktestExecutor |
| Portfolio | `src/ai_quant/portfolio/` | Positions + equity history |
| Analysis | `src/ai_quant/analysis/` | PerformanceAnalyzer |
| Reporting | `src/ai_quant/reporting/` | HTML Dashboard, risk report, multi-stock report |
| Engine | `src/ai_quant/engine/` | BacktestEngine coordination |

## Built-in Plugins

| Type | Names |
|------|-------|
| data_source | `csv`, `sqlite`, `t3time_csv`, `multistock_csv` |
| strategy | `ma_cross`, `rsi`, `bollinger`, `t3time` |
| risk_manager | `simple`, `full` |
| executor | `backtest` |
| analyzer | `performance` |

## Local Assets (gitignored)

| Path | Content |
|------|---------|
| `data/*.db` | HS300 SQLite daily/minute |
| `data/embeddings/` | T3Time h5 embeddings |
| `models/*.pth` | T3Time checkpoint |
| `output/` | HTML/JSON reports |

## Common Tasks

### Add Custom Strategy
1. Inherit `Strategy` from `ai_quant.strategy.base`
2. Implement `on_bar(bar)`
3. Call `emit_signal(direction, strength)`
4. Register in `registry.py`

### Add Custom Data Source
1. Inherit `DataSource` from `ai_quant.data.base`
2. Implement `load()` publishing `BarEvent`s
3. Register in `registry.py`

### Run Backtest
```python
from ai_quant.config import load_config_from_yaml
from ai_quant.engine import BacktestEngine

config = load_config_from_yaml("src/ai_quant/config.yaml")
engine = BacktestEngine(config)
engine.setup()
results = engine.run()
```

### Generate Visualization
Set `output.html_report` in YAML; Engine calls `reporting.generate_html_report`.
See [docs/VISUALIZATION.md](docs/VISUALIZATION.md).

## Documentation

```
docs/
├── ARCHITECTURE.md
├── TECHNICAL_DOC.md
├── VISUALIZATION.md
├── CORE_BELIEFS.md
├── STYLE_GUIDE.md
└── TESTING.md
```

## Tool Chain

| Tool | Purpose | Command |
|------|---------|---------|
| ruff | Lint + Format | `ruff check .` / `ruff format .` |
| mypy | Type Check | `mypy src/` |
| pytest | Testing | `pytest tests/` |

---

*This file is maintained by AI agents.*
*Last updated: 2026-08-30*

---
> Source: [itriones/AI-Quant-Framework](https://github.com/itriones/AI-Quant-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-01 -->
