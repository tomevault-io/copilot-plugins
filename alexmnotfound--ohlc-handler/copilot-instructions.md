## ohlc-handler

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# First-time setup (build Docker images, start Postgres, init schema, backfill all tickers)
make setup

# Start / stop the full stack
make up
make down
make logs

# Run the API locally (without Docker)
uvicorn api:app --reload --host 0.0.0.0 --port 8000

# Initialize (or migrate) the DB schema — safe to re-run, uses IF NOT EXISTS
make init-db

# Backfill all tickers and timeframes
make backfill

# Backfill a single ticker, optionally one timeframe
make backfill-ticker TICKER=BTCUSDT
make backfill-ticker TICKER=BTCUSDT TIMEFRAME=4h

# Extend history backwards from a start date
make extend-backfill START=2024-01-01 TICKER=BTCUSDT TIMEFRAME=1h

# Run processor directly with fine-grained control
python processor.py --ticker BTCUSDT --timeframe 1h --indicators ema
python processor.py --ticker BTCUSDT --skip-ohlc --indicators rsi
# --indicators choices: all | ema | rsi | obv | pivot | ce | patterns | daily_smma

# Connect to the database
make psql

# Clean Python cache files
make clean
```

## Architecture

### Data flow

```
Binance API → processor.py (CLI) / api.py (POST /update/…)
    → BinanceClient.get_klines()
    → DBHandler.save_klines()           → ohlc_data table
    → IndicatorCalculator / RSI / OBV / CE / Pivot / SMMA calculators
        each reads from ohlc_data, writes to its own table
    → api.py GET /ohlc/{symbol}/{timeframe}
        reads ohlc_data + all indicator tables, merges into response
```

### Key modules

- **`config.py`** — All tunables as dataclasses (`MarketConfig`, `APIConfig`, `DatabaseConfig`, `LoggingConfig`). `TICKERS`, `TIMEFRAMES`, indicator periods, `LOOKBACK_DAYS`, `BATCH_SIZES`, and `UPDATE_INTERVALS` all live here. The only env-driven config is DB credentials and `BINANCE_API_URL`.
- **`core/binance_client.py`** — Async `aiohttp` client. One session per instance; caller must `await client.close()`.
- **`core/db_handler.py`** — `psycopg2` wrapper. Opens a new connection per instance; caller must call `.close()`. No connection pool. All timestamps stored as naive UTC (`TIMESTAMP WITHOUT TIME ZONE`).
- **`processor.py`** — CLI entry point. Orchestrates fetch → indicator calculation across all tickers/timeframes. Use for backfills and cron-driven bulk updates.
- **`api.py`** — FastAPI. Read endpoint: `GET /ohlc/{symbol}/{timeframe}`. Update endpoints: `POST /update/{symbol}/{timeframe}`, `POST /timeframe/{timeframe}/update` (for cron), `POST /update/{symbol}`, `POST /update`.
- **`indicators/`** — One calculator class per indicator (EMA, RSI, OBV, Chandelier Exit, Pivot, Candle Patterns, Daily SMMA). Each opens its own `DBHandler`, fetches full OHLC history from `ohlc_data`, computes the indicator on the full series, and saves to its dedicated table.

### Incremental update pattern

All indicator calculators accept `only_save_last_n: Optional[int]`. They always compute the indicator on the full history (needed for accurate EWM/RMA seeding), but only save the last N rows when doing an incremental API-triggered update. The threshold used in `api.py` is `INCREMENTAL_SAVE_THRESHOLD = 50`.

### Scheduling

There is no in-process scheduler. Cron on the host calls `POST /timeframe/{timeframe}/update` shortly after each candle close (UTC times). Use `?calculate_indicators=false` for data-only fetches.

### Database tables

All timestamps are `TIMESTAMP WITHOUT TIME ZONE` (naive UTC). Tables: `ohlc_data`, `ema_data`, `rsi_data`, `obv_data`, `ce_data`, `pivot_data`, `daily_smma_99`. Candle patterns are stored inline in `ohlc_data.candle_pattern`.

### Adding a new ticker

Append the symbol to `TICKERS` in `config.py`. On next update, the fetcher detects no existing candle and automatically backfills using `LOOKBACK_DAYS[timeframe]` from now.

### Adding a new indicator

1. Create `indicators/<name>_calculator.py` following the existing pattern (open `DBHandler`, read from `ohlc_data`, compute, save, close in `finally`).
2. Add the table to `db/create_tables.sql` and re-run `make init-db`.
3. Wire it into `processor.py` (`_run_ohlc_and_indicators`) and `api.py` (`_trigger_update_timeframe_impl` and the relevant GET endpoint).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexmnotfound) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
