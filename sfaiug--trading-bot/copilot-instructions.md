## trading-bot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Python cryptocurrency trading bot** for Binance Futures Testnet. It supports two trading modes:
1. **Pyramid Trading** - Hedge + pyramid scaling + trailing stop strategy
2. **Funding Rate Farming** - Delta-neutral funding payment collection

## Commands

### Run Live Trading
```bash
# Interactive mode (prompts for all settings)
python main.py

# Direct mode - Pyramid Trading
python main.py --mode trading --symbol SOLUSDT --threshold 10 --trailing 1.5 --pyramid 2 --size 1

# Direct mode - Funding Rate
python main.py --mode funding --symbol SOLUSDT --entry-funding 0.1 --exit-funding 0.02 --size 1
```

### Backtesting
```bash
# Basic backtest with different thresholds
python backtest.py --thresholds 0.5,1,2,3,5 --days 1095 --symbol SOLUSDT

# Pyramid strategy backtest (uses tick data)
python backtest_pyramid.py

# Grid optimization (full parameter sweep)
python optimize_pyramid_v2.py
python optimize_pyramid_v2.py --coins BTCUSDT,ETHUSDT --quick-test
```

### Data
```bash
# Tick data is cached to avoid re-downloading
# Cache location configurable via TICK_DATA_CACHE env var
# Default: ./cache/trades/
```

## Architecture

### Core Trading Flow
```
main.py (CLI/UI entry)
    ├── core/exchange.py (BinanceExchange - API wrapper)
    ├── strategies/pyramid_trading.py (PyramidTradingStrategy)
    ├── strategies/funding_rate.py (FundingRateStrategy)
    └── ui/dashboard.py (TradingDashboard - Rich terminal UI)
```

### Strategy Logic

**Pyramid Trading** (`strategies/pyramid_trading.py`):
1. Opens hedge (1 LONG + 1 SHORT at same entry price)
2. Losing side closes when price moves `threshold%` against it
3. Pyramid reference = price where losing side closed
4. Adds positions every `pyramid_step%` from pyramid reference
5. All positions close when profit drops `trailing%` from max profit

**Funding Rate** (`strategies/funding_rate.py`):
1. Monitors funding rate every 8 hours
2. Enters SHORT when funding > entry_threshold (longs pay shorts)
3. Enters LONG when funding < -entry_threshold (shorts pay longs)
4. Exits when funding drops below exit_threshold

### Backtesting System
- `backtest.py` - Simple hedge trailing backtest using Binance klines
- `backtest_pyramid.py` - Pyramid strategy backtest module
- `optimize_pyramid_v2.py` - Grid search optimizer with tick data
- `data/tick_data_fetcher.py` - Downloads trades from data.binance.vision, aggregates to VWAP
- `analytics.py` - Post-backtest analytics (Sharpe, drawdown, monthly breakdown)

### Key Classes
- **BinanceExchange** (`core/exchange.py`) - All API calls, hedge mode, quantity rounding
- **PositionManager** (`core/position_manager.py`) - Position tracking, max profit updates
- **StateManager** (`core/state_manager.py`) - Persists state for recovery
- **TradingDashboard** (`ui/dashboard.py`) - Live Rich terminal UI with panels

## Configuration

### Environment Variables (.env)
```
BINANCE_API_KEY=your_testnet_api_key
BINANCE_API_SECRET=your_testnet_api_secret
TICK_DATA_CACHE=./cache/trades  # optional
```

### Settings (config/settings.py)
- `SYMBOL` - Default trading pair
- `POSITION_SIZE` - Default position size
- `TESTNET = True` - Always uses testnet
- `PRICE_CHECK_INTERVAL` - Polling interval (seconds)

## Symbol Precision

`BinanceExchange.DEFAULT_PRECISION` defines step sizes:
- BTCUSDT: 0.001, ETHUSDT: 0.01, SOLUSDT: 1
- Minimum notional: BTC=$100, others=$5

## Output Files

Optimizer results saved to `logs/`:
- `{COIN}_pyramid_v2_grid.csv` - Full grid search results
- `{COIN}_pyramid_v2_top10.csv` - Top 10 configurations
- `{COIN}_monthly_breakdown.csv` - Monthly P&L
- `pyramid_v2_recommendations.csv` - Final recommendations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sfaiug) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
