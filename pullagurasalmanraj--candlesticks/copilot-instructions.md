## candlesticks

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**Candlesticks** is a full-stack algorithmic trading research platform for Indian markets (NSE/BSE via Upstox broker API). It combines:
- Real-time market data ingestion (Upstox WSS → Redis → WebSocket)
- A state-machine-based market phase identifier (~3900 lines, `backend/routes/strategy.py`)
- Walk-forward ML pipeline (LightGBM binary + regressor, `backend/routes/ml.py`)
- Interactive backtesting and paper trading UI (React, `frontend/src/pages/LstmPredictor.jsx`)

## Running the Project

```bash
# Terminal 1: Flask backend (port 8000)
cd backend
python app.py

# Terminal 2: Async WebSocket server (port 9000)
cd backend
python wsserver.py

# Terminal 3: Frontend dev server (port 5173)
cd frontend
npm run dev
```

## Frontend Commands

```bash
cd frontend
npm install          # install deps
npm run dev          # dev server with HMR
npm run build        # production build → dist/ (served by Flask in prod)
npm run preview      # preview production build
```

## Database Setup

```bash
# Apply schema once against PostgreSQL (db: candleticks)
psql -U postgres -d candleticks -f backend/migration.sql
```

## Architecture

### Backend (`backend/`)

- **`app.py`** — Flask entry point (port 8000). Registers all blueprints, syncs Upstox instruments on startup, updates India VIX.
- **`wsserver.py`** — Separate async WebSocket server (port 9000). Connects to Upstox WSS, decodes protobuf ticks, publishes to Redis, broadcasts to frontend clients.
- **`config.py`** — All configuration (DB, Redis, Upstox API keys, paths).
- **`routes/strategy.py`** (~3900 lines) — Core state machine identifying market phases: `BULL_TREND`, `BEAR_TREND`, `GAP_UP`, `GAP_DOWN`, `IMPULSE_BULL`, `REJECTION`, `COMPRESSION`, `BALANCE_CHOP`, etc. Versions 23–27 fixed exit logic and cost viability gates.
- **`routes/ml.py`** (~1300 lines) — LightGBM pipeline: feature engineering (HTF join, prior phases), walk-forward validation, per-phase threshold tuning, model persistence to `models/*.pkl`.
- **`routes/indicators.py`** — NumPy-vectorized indicators (SMA, EMA, ATR, RSI, MACD, Bollinger, Supertrend, VWAP, OBV, ORB). 5–8x faster than `ta` library. Supertrend is the only non-vectorized part (band-dependency).
- **`routes/live.py`** — Real-time candle worker that consumes Redis ticks and assembles OHLCV bars.

### Frontend (`frontend/src/`)

- **`pages/LstmPredictor.jsx`** (~74 KB) — Main ML UI with sub-cards: `SearchTrainCard`, `PaperTradingCard`, `OfflineLabelingCard`, `EquityCurveCard`, `RulePerformanceCard`.
- **`hooks/useWebSocketPrices.js`** — Subscribes to port 9000 for live ticks.
- **`hooks/useInstrumentSearch.js`** — Fuzzy search via Fuse.js.
- **`context/ThemeContext.jsx`** — Aurora design system (dark/light toggle).
- **`services/`** — API client wrappers for candles, indicators, subscriptions.

### Data Flow

```
Upstox WSS → wsserver.py → Redis pub/sub → live.py (candle assembly) → PostgreSQL
                                         ↘ WebSocket broadcast → Frontend charts
```

### Database (PostgreSQL `candleticks`)

Key tables: `candles`, `strategy_outcomes`, `market_context`, `phase_params`, `ml_model_runs`, `paper_trade_runs`, `paper_trades`.

## Design System (Aurora Theme)

Defined in `frontend/tailwind.config.js` and `frontend/src/context/ThemeContext.jsx`.

- **Dark bg**: `#060b18` → `#0d1526` → `#132038`
- **Accents**: `#4f9eff` (blue), `#00e676` (bullish), `#ff5252` (bearish), `#ffd54f` (gold)
- **Fonts**: Syne (headings), DM Sans (body), JetBrains Mono (prices/tickers)
- **Layout**: Navbar 60px, Sidebar 240px (64px collapsed), card radius 12px

## Key Patterns

- Backend blueprints are registered in `app.py`; each route file exports a `*_bp` Blueprint.
- Upstox instrument keys use the format `NSE_EQ|BAJAJFINSV`; mapping helpers are in `utils/symbol_map.py`.
- Redis is used for both tick streaming (pub/sub) and token caching. DB index is `/10`.
- ML models are persisted as `.pkl` files under `backend/models/`, named by symbol and timeframe.
- The frontend build output (`dist/`) is served statically by Flask in production — `vite.config.js` sets a relative base path for this.

---
> Source: [pullagurasalmanraj/candlesticks](https://github.com/pullagurasalmanraj/candlesticks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
