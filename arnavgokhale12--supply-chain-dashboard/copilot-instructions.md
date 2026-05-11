## supply-chain-dashboard

> Supply Chain Dashboard - A web application that tracks supply-chain stress using a composite index built from macroeconomic indicators, with market overlays showing how stocks and sectors perform under different supply chain regimes.

# CLAUDE.md

## Project Overview

Supply Chain Dashboard - A web application that tracks supply-chain stress using a composite index built from macroeconomic indicators, with market overlays showing how stocks and sectors perform under different supply chain regimes.

## Tech Stack

- **Backend:** FastAPI, SQLAlchemy, Alembic, SQLite
- **Frontend:** Streamlit, Pandas
- **Data Sources:** FRED API, NY Fed (Excel), yfinance (market data)

## Project Structure

```
backend/app/
├── core/config.py       # Settings via pydantic (API keys, DB URL)
├── db/                  # SQLAlchemy session, base, deps
├── models/
│   ├── series.py            # Economic indicator metadata
│   ├── observation.py       # Time series data points
│   ├── indicator_config.py  # Dynamic indicator settings (weights, signs)
│   ├── market_series.py     # Stock/ETF metadata
│   ├── market_price.py      # Daily price data
│   └── regime_return.py     # Precomputed regime statistics
├── routers/
│   ├── health.py            # Health check
│   ├── series.py            # Series endpoints
│   ├── observations.py      # Observation endpoints
│   ├── composite.py         # Composite index endpoints
│   └── market.py            # Market data endpoints
├── services/
│   ├── seed.py              # Initialize series + configs
│   ├── fred.py              # FRED API client
│   ├── ingest_gscpi.py      # NY Fed GSCPI
│   ├── ingest_cass.py       # Cass Freight Index
│   ├── ingest_retailirsa.py # Retail Inventories
│   ├── ingest_fred_indicators.py  # ISM, Mfg Orders, Wholesale
│   ├── ingest_baltic_dry.py       # Baltic Dry Index (Quandl)
│   ├── ingest_market_data.py      # Stocks/ETFs via yfinance
│   ├── composite_calculator.py    # Dynamic N-indicator composite
│   └── regime_analyzer.py         # Regime-conditional returns
└── main.py              # FastAPI app with CORS

frontend/app.py          # Streamlit dashboard with market overlays
```

## Key Commands

```bash
# Run backend (from project root)
source venv/bin/activate
uvicorn backend.app.main:app --reload

# Run frontend
cd frontend && streamlit run app.py

# Database migrations
alembic upgrade head

# Data ingestion - Supply Chain Indicators
python -m backend.app.services.seed                    # Initialize metadata
python -m backend.app.services.ingest_gscpi            # NY Fed GSCPI
python -m backend.app.services.ingest_cass             # Cass Freight
python -m backend.app.services.ingest_retailirsa       # Retail Inventories
python -m backend.app.services.ingest_fred_indicators  # ISM, Mfg Orders, Wholesale

# Data ingestion - Market Data
python -m backend.app.services.ingest_market_data      # Stocks/ETFs (yfinance)
python -m backend.app.services.regime_analyzer         # Compute regime returns
```

## Environment Variables

- `DATABASE_URL` - SQLite connection string (default: `sqlite:///./scdash.db`)
- `FRED_API_KEY` - Required for FRED API calls
- `QUANDL_API_KEY` - Optional, for Baltic Dry Index
- `ALPHA_VANTAGE_API_KEY` - Optional, not used (switched to yfinance)
- `API_BASE` - Backend URL for frontend (default: `http://127.0.0.1:8000`)

## API Endpoints

### Supply Chain
- `GET /health` - Health check
- `GET /series/{series_id}/latest` - Latest observation for a series
- `GET /observations?series_id=&start=&end=` - Historical observations
- `GET /v1/composite/latest` - Current composite score with all indicators
- `GET /v1/composite/history` - Historical composite index
- `GET /v1/composite/indicators` - List configured indicators

### Market Data
- `GET /v1/market/series` - List all stocks/ETFs
- `GET /v1/market/prices/{symbol}?days=365` - Historical prices
- `GET /v1/market/regime-returns?regime=&theme=` - Regime statistics
- `GET /v1/market/current` - Latest prices with regime context

## Supply Chain Indicators

| ID | Name | Source |
|----|------|--------|
| gscpi | Global Supply Chain Pressure Index | NY Fed |
| retailirsa | Retail Inventories/Sales Ratio | FRED |
| cass | Cass Freight Index | FRED |
| ism_supplier | Manufacturing Employment | FRED |
| mfg_new_orders | Manufacturing New Orders | FRED |
| wholesale_ratio | Wholesale Inventories/Sales | FRED |
| baltic_dry | Baltic Dry Index | Quandl |

## Market Symbols

| Symbol | Type | Theme |
|--------|------|-------|
| SPY | index | - |
| XLI, XLY, XLK, SMH | etf | industrials, consumer, technology, chips |
| NVDA, TSM, AVGO | stock | chips |
| AMZN | stock | logistics |
| COST, WMT | stock | retail |

## Composite Index Logic

- Dynamically combines N indicators via `IndicatorConfig` table
- Each indicator has: `weight`, `invert_sign`, `include_in_composite`
- Z-score normalization using 36-month rolling window
- Weighted average: `sum(z * weight) / sum(weight)`
- Regime classification: `low` (<-0.5), `normal` (-0.5 to 0.5), `elevated` (0.5 to 1.5), `crisis` (>=1.5)

## Database Models

### Supply Chain
- **Series:** id (PK), name, frequency, source, url
- **Observation:** id (PK), series_id (FK), date, value
- **IndicatorConfig:** series_id (PK), include_in_composite, weight, invert_sign, display_order

### Market
- **MarketSeries:** symbol (PK), name, series_type, theme
- **MarketPrice:** id (PK), symbol (FK), date, close, adjusted_close, volume
- **RegimeReturn:** id (PK), symbol (FK), regime, avg_monthly_return, std_monthly_return, sample_count

## Frontend Layout

```
┌─────────────────────────────────────────────────────┐
│              STATUS BANNER (regime + score)          │
├─────────────────────────┬───────────────────────────┤
│   LEFT (60%)            │   RIGHT (40%)             │
│   - Composite Score     │   - Market Impact Panel   │
│   - All Indicators      │   - S&P 500 + ETFs        │
│   - History Chart       │   - Regime context        │
├─────────────────────────┴───────────────────────────┤
│              KEY STOCKS BY THEME                     │
│  [Semiconductors]    [Retail]    [Logistics]        │
└─────────────────────────────────────────────────────┘
```

---
> Source: [arnavgokhale12/supply-chain-dashboard](https://github.com/arnavgokhale12/supply-chain-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
