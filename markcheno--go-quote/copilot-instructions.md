## go-quote

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

go-quote is a Go library and CLI tool for downloading historical price quotes. It supports:
- Tiingo API (stocks, daily/intraday data) - requires TIINGO_API_TOKEN
- Tiingo Crypto API (cryptocurrency data)
- Coinbase API (cryptocurrency exchange data)

The project has no external dependencies beyond the Go standard library.

## Build and Test

```bash
# Build the CLI
go build ./quote

# Run tests
go test

# Install CLI globally
go install github.com/markcheno/go-quote/quote@latest

# Run CLI
quote -help
```

## Architecture

### Core Components

**quote.go** - Main library with ~1800 lines implementing:
- `Quote` struct: Single symbol with Date/OHLCV slices
- `Quotes` array: Multiple symbols
- Data source functions: `NewQuoteFromTiingo()`, `NewQuoteFromTiingoCrypto()`, `NewQuoteFromCoinbase()`
- CSV/JSON parsing and output: `NewQuoteFromCSV()`, `WriteCSV()`, `WriteJSON()`, etc.
- Market list fetchers: `NewMarketList()`, `NewEtfList()` - downloads symbol lists from NASDAQ API and FTP
- Update mode: `UpdateFileTiingo()` - in-place CSV updates with backfill

**quote/main.go** - CLI wrapper (~470 lines):
- Flag parsing for all options (years, period, source, format, etc.)
- Symbol resolution from files, markets, or arguments (supports wildcards in -infile)
- Two output modes: individual files per symbol or all-in-one file (-all=true)
- Update mode entry point

### Data Flow

1. **Symbol Resolution**: CLI resolves symbols from -infile (with wildcard support), -markets flag, or command args
2. **Fetch**: Library makes HTTP requests to Tiingo/Coinbase with proper headers and rate limiting
3. **Transform**: API JSON responses are parsed into Quote structs with OHLCV data
4. **Output**: Data written as CSV (default), JSON, Highstock format, or Amibroker format

### Update Mode Architecture

The `-update` flag enables incremental CSV updates for Tiingo data (quote.go:596-1064):

- **Single-symbol CSVs**: Infers symbol from filename (e.g., spy.csv → SPY)
- **Multi-symbol CSVs**: Preserves original symbol order from input file
- **Backfill window**: `-backfill-days` (default 10) rewrites overlap period to catch adjustments
- **Corporate action detection**: Detects splits/dividends in backfill window; if `-full-redownload-on-ca` is set, re-fetches entire history for that symbol
- **Concurrency**: `-concurrency` controls parallel symbol fetching (multi-symbol only)
- **Rate limiting**: Respects global `Delay` variable (set via `-delay` flag) using ticker-based limiter across all workers

Implementation uses two-pass approach:
1. First pass: Scan file to determine date ranges and symbol order
2. Second pass: Rewrite .tmp file with preserved data before cutoff + new/updated data from API

## Key Implementation Details

### Rate Limiting
- `quote.Delay` (global variable) controls milliseconds between requests
- CLI sets this via `-delay` flag (default 100ms)
- Update mode uses `time.Ticker` for global rate limiting across concurrent workers

### API Interactions
- **Tiingo**: Uses Authorization header with token, supports date ranges and resample frequencies
- **Coinbase**: Public API, fetches in 200-bar chunks with pagination
- **NASDAQ API**: Fetches market screeners and symbol lists via JSON API
- **FTP**: ETF list via anonymous FTP to ftp.nasdaqtrader.com

### Testing Strategy
- quote_test.go contains unit tests for CSV parsing and update mode
- Update tests use `tiingoFetch` variable indirection for mocking
- Tests use `t.TempDir()` for isolated file operations

### Period Handling
Period constants map user input to API parameters:
- Tiingo daily: d, w, m
- Tiingo crypto: 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 8h, 12h, d
- Coinbase: 1m, 5m, 15m, 30m, 1h, d, w (mapped to granularity in seconds)

### CSV Format
Single-symbol: `datetime,open,high,low,close,volume`
Multi-symbol: `symbol,datetime,open,high,low,close,volume`
Date format: `2006-01-02 15:04` (time is always 00:00 for daily data)

## Development Notes

- Go version: 1.22+ (per go.mod)
- No external dependencies
- All HTTP clients use 10-second timeout (`ClientTimeout` constant)
- User agents are randomized from a pool (Chrome, Firefox, Safari, Edge) for NASDAQ API requests
- Precision: 2 decimals for stocks, 8 decimals for crypto (BTC/ETH/USD symbols)

---
> Source: [markcheno/go-quote](https://github.com/markcheno/go-quote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
