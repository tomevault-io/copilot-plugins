## opes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Subagent Rules
- Do NOT spawn Explore for simple file lookups or single-file reads — do it directly
- Only use Explore for broad codebase searches across many files

## Instructions
When creating a new function make a concise comment with what it does

Be critical do not default to agree with me. Try to give more ideas on what my be best for the project

# Python dependencies
pip install -r requirements.txt
```

See `Docs/start_dev_server.txt` for the dev setup.

## Architecture

**Opes** is a stock analysis app — React 19 SPA frontend + Python serverless backend deployed on Vercel.

### Frontend (`src/`)

Three views managed by `App.jsx`:
- **Analyze** — User enters a stock symbol; app fetches deep analysis (technical indicators, ML prediction, fundamentals)
- **TopStocks** — Displays top 5 ranked buy-rated stocks from a hardcoded 18-stock watchlist
- **Portfolio** — Mock holdings with sparklines; no backend, pure local data from `src/services/portfolio.js`

Services (`src/services/`):
- `api.js` — Calls `/api/analyze`, caches in localStorage (daily expiry, cache version 2, stale-cache fallback)
- `topStocks.js` — Calls `/api/topstocks`, same daily localStorage cache pattern; falls back to `topStocksMockData` when API unavailable
- `portfolio.js` — Pure mock data, no API calls

### Backend (`api/`)

Two Vercel Python serverless handlers:

**`analyze.py` — `GET /api/analyze?symbol=X`**
- Fetches in parallel (3 threads): Yahoo Finance 1yr prices, current quote, Yahoo Finance quoteSummary fundamentals
- Computes: SMA (20/50), EMA (12/26), RSI (14), MACD (12/26/9), Bollinger Bands (20), OBV
- Trains a RandomForestClassifier (100 estimators, depth 5) on ~230 training samples from 1yr of data
- ML features: 5-day returns/volume/HL ratio, price position, RSI proxy, SMA crossover, Bollinger %B, 52-week high distance, ROC deceleration
- Label threshold: >0.5% forward return (not just >0)
- Neutral band: returns "Neutral / Hold" when model confidence is 0.45–0.55
- Walk-forward validation: 3-fold expanding (train 60/70/80%, test 70/80/90%); averaged accuracy returned as `validation_accuracy`; caveat appended to reasoning when accuracy < majority-class baseline + 5pp
- Composite quality score: fundamentals 40% + ML confidence 40% + momentum 20%

**`topstocks.py` — `GET /api/topstocks`**
- Processes hardcoded `WATCHLIST` (18 large-cap stocks: NVDA, META, AMZN, AAPL, GOOGL, etc.)
- Concurrent price fetch (uncapped threads) + concurrent fundamentals fetch (uncapped — Yahoo Finance has no rate limit); fundamentals fetch includes `assetProfile` module for sector data
- Same ML feature set, 3-fold walk-forward, and 4-pillar scoring as `analyze.py`
- Computes per-sector median PE dynamically from watchlist fundamentals via `_compute_sector_medians()` — no stale hardcoded values
- Returns top 5 bullish stocks by quality score; pads with non-bullish if fewer than 5
- Neutral-direction stocks are excluded from top picks

**`api/utils/scoring.py`** — shared scoring logic imported by both handlers:
- `compute_fundamentals_score(fund, sector_medians=None)` — uses PEG ratio (PE / earnings growth%) when `earnings_growth` available; falls back to sector-relative PE (via `SECTOR_PE_MEDIANS` or caller-supplied `sector_medians` dict) with a forward PE acceleration bonus; absolute PE thresholds only for unknown sectors
- `SECTOR_PE_MEDIANS` — hardcoded trailing PE medians by sector (2024-2025); used as fallback in `analyze.py`; overridden by dynamic medians in `topstocks.py`
- `compute_momentum_score(closes)` — includes overextension penalty when price exceeds Bollinger upper band
- `compute_quality_score(ml_confidence, fund_score, momentum_score)`
- `quality_score_to_label(score, direction)`

**`api/dev_server.py`** — local development only; routes `/api/analyze` and `/api/topstocks` to the appropriate handler modules.

### Data Sources

| Source | Auth | Usage |
|--------|------|-------|
| Yahoo Finance Chart API (`/v8/finance/chart`) | None | Prices (1yr OHLCV), current quote |
| Yahoo Finance quoteSummary (`/v10/finance/quoteSummary`) | None | PE, ROE, EPS, profit margin, earnings growth, sector, 52-week high/low, beta |

All fundamentals now come from Yahoo Finance. 

### Deployment

`vercel.json` routes `api/*.py` (top-level only, not `api/utils/`) to Vercel's Python runtime. Static frontend served from `/dist`. The ML model is retrained on every backend request — no persistence.

---
> Source: [cghubble8/Opes](https://github.com/cghubble8/Opes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
