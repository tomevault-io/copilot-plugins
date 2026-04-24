## nifty-indices

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nifty 100 Index Tracker - A Python application that helps retail investors replicate the Nifty 100 index by calculating proportional buy orders based on investment amounts. The system fetches real-time stock prices from NSE and generates portfolio allocation reports with equal weight distribution (1% per security).

## Common Development Commands

### Basic Workflow
```bash
# Install dependencies
pip install -r requirements.txt

# Download latest Nifty 100 constituents
python src/main.py --download-constituents

# Enhance securities data with prices and P/E ratios
python src/main.py --hydratedata data/ind_nifty100list_*.csv --output data/nifty100_enhanced_securities.csv

# Generate portfolio allocation
python src/main.py --amount 1100000 --securities data/nifty100_enhanced_securities.csv --exclusion data/nifty100_exclusions.csv

# Create sample data for testing
python src/main.py --create-sample
python src/main.py --create-exclusion-sample

# Run in debug mode
python src/main.py --amount 100000 --log-level DEBUG
```

### Testing (Currently no test framework is set up)
```bash
# Run application with test data
python src/main.py --amount 10000 --securities data/sample_securities.csv
```

## Code Architecture

### Core Structure
The application follows a service-oriented architecture with clear separation of concerns:

1. **Entry Point**: `src/main.py` - CLI interface and orchestration
2. **Data Models** (`src/models/`):
   - `security.py`: Security dataclass representing stocks
   - `portfolio.py`: Portfolio, Allocation, and Summary dataclasses
3. **Services** (`src/services/`):
   - `price_fetcher.py`: Fetches real-time prices from NSE via yfinance
   - `allocator.py`: Implements equal-weight portfolio allocation logic
   - `csv_handler.py`: Handles all CSV read/write operations
4. **Utilities** (`src/utils/`):
   - `validators.py`: Input validation (amounts, file paths)
   - `helpers.py`: Formatting and helper functions
5. **Configuration**: `config/settings.py` - Centralized settings

### Key Design Patterns
- **Service Pattern**: Each service handles a specific domain (prices, allocation, CSV)
- **Dataclass Models**: All data structures use Python dataclasses for type safety
- **Error Handling**: Graceful failures with retry mechanisms for API calls
- **Batch Processing**: Generates JSON output in batches for large portfolios

### Data Flow
1. Load Nifty 100 constituents from CSV or download from NSE
2. Apply optional exclusion filters (matched by Symbol and ISIN)
3. Fetch real-time prices using yfinance (with retry logic)
4. Calculate equal-weight allocations (1% per security)
5. Generate buy orders (whole shares only)
6. Output results in CSV/JSON format with summary statistics

### Important Implementation Details
- **Equal Weight Only**: Currently implements only equal weight allocation (1% per security)
- **Fractional Shares**: Rounds down to whole shares, tracks unallocated amounts
- **Price Fetching**: Uses yfinance with NSE tickers (append .NS suffix)
- **Error Recovery**: Continues processing if individual securities fail
- **Investment Limits**: ₹1,000 to ₹10 crores (defined in settings.py)

## Key Files to Understand

1. **config/settings.py**: All configuration constants and constraints
2. **src/services/allocator.py**: Core allocation algorithm implementation
3. **src/services/price_fetcher.py**: NSE integration and price fetching logic
4. **src/main.py**: Command-line argument parsing and workflow orchestration

## Development Notes

- No unit tests currently exist (planned for future phase)
- No linting or type checking configuration
- CSV is the primary data format (no database)
- All file paths should use absolute paths internally
- Logging can be enabled with --log-level DEBUG for troubleshooting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noelcurtis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
