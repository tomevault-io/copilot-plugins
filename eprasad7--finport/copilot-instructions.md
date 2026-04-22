## finport

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Running the Application
```bash
# Local development
python app.py

# Production (using gunicorn)
gunicorn app:app
```

### Installing Dependencies
```bash
pip install -r requirements.txt
```

### Testing
```bash
# Run API tests (from README)
./test_api.py

# Test with specific ticker
./test_api.py AAPL
```

## High-Level Architecture

This is a Flask-based financial data portal with the following architecture:

### Entry Points
- `app.py` - Main entry point that imports the Flask app from `src.main`
- `src/main.py` - Flask application setup, database initialization, and route registration

### Core Components

1. **Models** (`src/models/`)
   - **Security Models**: `Security`, `PriceData`, `FTDData`, `InstitutionalOwnership`, `OptionData`, `ETFHolding`
   - **User Models**: `User`, `Watchlist`, `WatchlistItem`, `UserSetting`, `Alert`
   - **Analytics Models**: `SwapCycle`, `VolatilityCycle`, `MarketCorrelation`, `TechnicalIndicator`
   - **API Integration Models**: `ApiProvider`, `ApiKey`, `ApiEndpoint`, `ApiCallLog`, `DataSyncLog`

2. **Services** (`src/services/`)
   - `polygon_service.py` - Integration with Polygon.io API for market data
   - `ftd_service.py` - Failure-to-Deliver data processing
   - `analytics_service.py` - Technical analysis and market analytics

3. **Routes** (`src/routes/`)
   - `user.py` - User management endpoints (`/api/users/*`)
   - `security.py` - Security/market data endpoints (`/api/securities/*`)

4. **Database**
   - SQLite database at `src/database/app.db`
   - Uses Flask-SQLAlchemy ORM
   - Tables auto-created on startup via `db.create_all()`

### API Structure
The API follows RESTful conventions with two main route groups:
- `/api/securities/*` - Market data, FTD analysis, technical indicators
- `/api/users/*` - User management, watchlists, alerts, settings

### Frontend
Static HTML files served from `src/static/`:
- `index.html` - Main entry page
- `dashboard.html` - Financial data dashboard

### Key Configuration
- Polygon API key is hardcoded in `src/main.py`
- Flask secret key is hardcoded in `src/main.py`
- Database URI points to local SQLite file
- CORS is enabled for all origins

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eprasad7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
