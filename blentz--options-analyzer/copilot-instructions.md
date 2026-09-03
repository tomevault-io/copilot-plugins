## options-analyzer

> This document provides context for AI agents working on this codebase.

# AGENTS.md - AI Agent Guidelines

This document provides context for AI agents working on this codebase.

## Project Overview

Options Trading Analyzer is a FastAPI web application that imports Fidelity CSV exports and provides analytics on options trading performance. It runs in a Podman container with WSL on Windows.

## Key Architecture Decisions

### Async Everything
- SQLAlchemy with `aiosqlite` for async database operations
- All database functions use `async/await`
- FastAPI's async request handlers

### Fidelity CSV Parsing
The CSV parser in `csv_import.py` handles Fidelity's non-standard format:
- Headers start at "Run Date," row (not first line)
- Columns are shifted (Quantity in "Price" column, Price in "Currency" column)
- Options symbols start with `-` (e.g., `-AAPL251121P150`)
- Description field contains readable contract info

### Position State Management
Positions are considered closed when:
1. Net quantity equals zero (bought back/sold to close), OR
2. Outcome is EXPIRED or ASSIGNED, OR
3. Contract expiration date has passed

### P&L Calculation
- `net_pnl`: Options premium only (sum of trade amounts)
- `underlying_pnl`: Stock P&L from assignments (linked UnderlyingTrade records)
- `total_pnl`: Combined (net_pnl + underlying_pnl)

Win/loss determination uses `total_pnl` to account for assignment outcomes.

## Common Tasks

### Adding a New Page
1. Add route in `app/main.py`
2. Create template in `templates/`
3. Add nav link in `templates/base.html`

### Adding Analytics
1. Add dataclass in `app/services/analytics.py`
2. Create async function to query positions
3. Use in route handler

### Adding Charts
1. Create chart function in `app/charts.py`
2. Return `(script, div)` tuple from `components()`
3. Pass to template, render with `| safe` filter

### Modifying Database Schema
1. Update models in `app/models.py`
2. Delete `data/options.db` to recreate (or write migration)
3. Re-import CSV data

## Testing Changes

```bash
# Rebuild container after code changes
podman build -t options-analyzer .

# Restart with new image
podman rm -f options-analyzer
podman run -d --name options-analyzer -p 8000:8000 -v ./data:/app/data options-analyzer

# Check logs
podman logs options-analyzer

# Test endpoints
curl http://localhost:8000/api/stats
```

## File Purposes

| File | Purpose |
|------|---------|
| `app/main.py` | FastAPI app, all HTTP routes |
| `app/models.py` | SQLAlchemy ORM models |
| `app/database.py` | DB engine and session factory |
| `app/charts.py` | Bokeh chart generation |
| `app/services/csv_import.py` | Fidelity CSV parsing, trade/position creation |
| `app/services/analytics.py` | Statistics queries and aggregations |
| `app/services/risk_analysis.py` | Options payoff calculations |
| `app/services/price_service.py` | Yahoo Finance API client with caching |
| `templates/base.html` | Base template with nav, styles |
| `templates/dashboard.html` | Main dashboard with stats and charts |
| `templates/positions.html` | Position list with filters |
| `templates/risk.html` | Risk analysis page |
| `templates/import.html` | CSV upload form |

## Gotchas

1. **Fidelity CSV columns are misaligned** - Don't trust column names, use the shifted mapping in `parse_csv_content()`

2. **Yahoo Finance rate limiting** - Always include User-Agent header, use 60-second cache

3. **Expired positions without trades** - Contracts past expiration date are auto-marked as EXPIRED even without explicit trade record

4. **Container rebuilds required** - Code changes require `podman build` and container restart

5. **Database path** - Hardcoded to `/app/data/options.db` in container, mount volume to persist

## Dependencies

Managed via `uv` — see `pyproject.toml` for the full pinned set, `uv.lock`
for the resolved tree. Sync with `uv sync --extra dev`.

Core dependencies:
- `fastapi` + `uvicorn` - Web framework
- `sqlalchemy` + `aiosqlite` - Async ORM (SQLite with WAL mode)
- `bokeh` - Interactive charts
- `pandas` - Data manipulation
- `httpx` - Async HTTP client (Yahoo Finance, with retry)
- `jinja2` - Templates
- `python-multipart` - File uploads
- `playwright` - StockNear scraper (Firefox, persistent context)
- `scipy` - Probability functions (norm.ppf)
- `yfinance` - Options chains fallback

Dev dependencies (`uv sync --extra dev`):
- `pytest` + `pytest-asyncio` - Tests live in `tests/`

## Running tests

```bash
uv run pytest          # all tests
uv run pytest -v       # verbose
uv run pytest tests/test_risk_math.py  # single file
```

---
> Source: [blentz/options-analyzer](https://github.com/blentz/options-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
