## algotrading

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an algorithmic trading system that executes automated trades on the Alpaca platform using technical analysis strategies. The system runs continuously during market hours, managing both long and short positions with automatic profit-taking and stop-loss logic.

## Environment Setup

### Required Environment Variables (.env file)
- `ALPACA_API_KEY_ID_LIVE` - Live trading API key
- `ALPACA_API_SECRET_KEY_LIVE` - Live trading secret key
- `ALPACA_API_KEY_ID` - Dashboard/monitoring API key
- `ALPACA_API_SECRET_KEY` - Dashboard/monitoring secret key
- `ALPACA_API_KEY_ID_PAPER` - Paper trading API key (for screener)
- `ALPACA_API_SECRET_KEY_PAPER` - Paper trading secret key
- `PORTFOLIO_STARTING_VAL` - Starting portfolio value for relative equity calculations

### Installation
```bash
pip install -r requirements.txt
```

## Architecture

### Core Components

**[main.py](main.py)** - Live trading execution engine
- Runs the main trading loop that executes every 60 seconds
- Implements position management with automatic profit-taking (8% gain) and stop-loss (-2% loss)
- Manages both long and short positions
- Uses signal strength to calculate position sizing (multiplier of 6x)
- Minimum position size is 1/150th of buying power
- Resets daily state when market closes (stocksBoughtToday, stocksShortedToday lists)
- Trading logic:
  - Opens positions based on strategy signals with strength threshold (1/(multiplier*150))
  - Adds to positions every 30 iterations if signal persists
  - Closes positions when signals reverse
  - Prevents re-entry after stop-loss (stocksNotToLong/stocksNotToShort lists)

**[backend.py](backend.py)** - FastAPI server for portfolio monitoring
- Provides REST endpoints for dashboard data with 5-minute caching
- Endpoints: `/portfolio`, `/positions`, `/portfolio_history_day`, `/portfolio_history_week`, `/portfolio_history_month`, `/starting_balance`
- Uses paper trading API credentials (not live credentials)
- Deployed on Render at https://algotrading-8wcz.onrender.com

**[dashboard.py](dashboard.py)** - Streamlit portfolio visualization
- Real-time portfolio tracking with equity curves and position details
- Displays relative equity (normalized to starting balance)
- Shows position-level P&L percentages and equity allocation
- Duration views: 1D (5min bars), 1W (15min bars), 1M (1D bars)
- Run with: `streamlit run dashboard.py`

**[FidelityScreener.py](FidelityScreener.py)** - Stock screener utility
- Ranks stocks by signal strength using the current strategy
- Uses paper trading credentials to fetch data
- Outputs sorted list of tickers with signal strength percentages

### Strategy System

**Strategies/** directory contains pluggable trading strategies. Each strategy:
- Takes a DataFrame with OHLCV data as input
- Returns DataFrame with added columns: `Signal` (bool), `SignalStrength` (float), `Position` (lagged signal)
- Current strategies:
  - **SMAStrat**: Simple Moving Average crossover (50/200 periods)
  - **EWMAStrat**: Exponential Weighted Moving Average crossover (50/200 periods) - currently active in [main.py:224](main.py#L224)

To switch strategies, change the strategy parameter in [main.py:224](main.py#L224) from `EWMAStrat` to `SMAStrat` or implement a new strategy following the same interface.

### Data Flow

1. [main.py](main.py) `getData()` fetches 399 1-minute bars from Alpaca
2. Strategy function processes bars and generates signals with strength values
3. `place_order()` calculates position sizes based on signal strength and buying power
4. Orders are submitted via Alpaca REST API (market orders, day duration)
5. [backend.py](backend.py) caches portfolio data from Alpaca for dashboard consumption
6. [dashboard.py](dashboard.py) fetches from backend and renders visualizations

### Position Management Logic

**Order Actions:**
- `buy` - Open long position
- `sell` - Close long position
- `sellsh` - Open short position (converted from sell side internally)
- `buyc` - Close short position (converted from buy side internally)

**Risk Management:**
- Fractional shares used when available, integer shares otherwise
- Daily tracking prevents over-trading (stocksBoughtToday/stocksShortedToday)
- Positions > 1/8th equity can be closed for profit/loss regardless of daily tracking
- Market closure triggers daily state reset

## Development Commands

### Running the Trading System
```bash
# Live trading (uses ALPACA_API_KEY_ID_LIVE credentials)
python main.py
```

### Running the Dashboard
```bash
# Start backend API (if not already deployed on Render)
uvicorn backend:app --reload

# Start dashboard (connects to deployed backend)
streamlit run dashboard.py
```

### Stock Screening
```bash
# Rank stocks by current signal strength
python FidelityScreener.py
```

## Deployment Notes

- Main trading bot is designed to run on AWS EC2 t2.micro instance
- Backend API is deployed on Render (https://algotrading-8wcz.onrender.com)
- Dashboard can run locally or be deployed separately
- AWS setup instructions: https://docs.google.com/document/d/1GChJQ5csmOPUliuxAmOvEmVO5-U8iIXSoTiyjfx6w5Y/edit?tab=t.0#heading=h.vmu8u9tjt96e

## Important Codebase Conventions

- Signal strength is normalized relative to price (calculated as ratio difference divided by base value)
- Position sizing uses signal strength with a hardcoded multiplier (currently 6 in [main.py:127](main.py#L127))
- All timestamps are converted to US/Eastern timezone for display
- Equity values in dashboard are normalized to starting balance for relative performance tracking
- The ticker list in [main.py:189-223](main.py#L189-L223) defines which stocks are actively traded

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LZimm1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
