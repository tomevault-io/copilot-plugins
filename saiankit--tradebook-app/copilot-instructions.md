## tradebook-app

> This project is a CLI-based portfolio management application designed to track stock trades, calculate P&L (Realized and Unrealized), and provide performance metrics like XIRR.

# Tradebook App - Project Documentation

This project is a CLI-based portfolio management application designed to track stock trades, calculate P&L (Realized and Unrealized), and provide performance metrics like XIRR.

## Architecture & Components

The application follows a modular structure where concerns are separated into distinct Python modules:

### 1. Core Entry Point (`app.py`)
- Provides a CLI interface using `argparse`.
- Commands:
  - `import <file_path>`: Imports trades from a CSV file.
  - `dashboard`: Displays the portfolio status for **open positions**, including invested value, current value, unrealized P&L, and XIRR.
  - `realized`: Displays the history of realized profit/loss grouped by year.
  - `trades [--symbol SYMBOL]`: Displays a detailed log of all trades, optionally filtered by a specific stock symbol.

### 2. Data Models & Persistence (`models.py`)
- **Schema**: Uses SQLite (`tradebook.db`) to store trades.
- **Table**: `trades` (trade_id, symbol, isin, trade_date, exchange, trade_type, quantity, price, order_id, order_execution_time).
- **Primary Key**: Uses a composite primary key `(trade_id, trade_date, order_execution_time)` to uniquely identify executions and handle cases where brokers reuse `trade_id` across different sessions.
- **Functionality**: Handles database initialization, trade insertion (with `IGNORE` on duplicate records), and data retrieval.

### 3. Data Ingestion (`importer.py`)
- Reads trade data from CSV files using `pandas`.
- Maps CSV columns to the `Trade` dataclass.
- **Expected CSV Columns**: `symbol`, `isin`, `trade_date`, `exchange`, `trade_type`, `quantity`, `price`, `trade_id`, `order_id`, `order_execution_time`.
- **Date Formats**:
  - `trade_date`: Expected in `YYYY-MM-DD` format.
  - `order_execution_time`: Stored as a string from the CSV.

### 4. Portfolio Logic (`portfolio.py`)
- **Sorting**: Trades are retrieved from the database sorted by `trade_date ASC` and `order_execution_time ASC` to ensure FIFO calculations are chronologically accurate.
- **FIFO Calculation**: Implements symmetric FIFO logic for both long and short positions.
  - Matches `sell` trades against `buy_queue` (closing long).
  - Matches `buy` trades against `sell_queue` (closing short).
  - Handles intraday and multi-day short trades.
- **Market Data**: Uses `yfinance` to fetch real-time (or latest) prices for symbols. Symbols are suffixed with `.NS` (NSE) or `.BO` (BSE) for compatibility.
- **Metrics**: 
  - **P&L**: Calculates both realized (by year) and unrealized gains. For short positions, unrealized P&L is calculated as `(sell_price - current_price) * quantity`.
  - **XIRR**: Uses the `pyxirr` library to calculate the Extended Internal Rate of Return for both individual symbols and the overall portfolio.
    - **Open Positions Focus**: Both symbol-level and overall portfolio XIRR only include the cash flows for **currently held (open)** positions. This ensures the performance metrics reflect active capital and are not distorted by historical gains/losses.
    - **Cash Flows**: Derived from the FIFO purchase dates/prices of held shares (outflows) and their current market value (inflow).

### 5. Visualization (`dashboard.py`)
- Utilizes the `rich` library for high-quality console output.
- Displays:
  - A table of current holdings with quantity, average price, invested value, LTP, current value, unrealized P&L, and XIRR.
  - Overall portfolio summary (Total Invested, Total Current Value, Total Unrealized P&L, Overall XIRR).
  - Conditional formatting: Unrealized P&L and XIRR are colored green (positive) or red (negative).

## Key Dependencies
- `pandas`: Data manipulation and CSV parsing.
- `yfinance`: Fetching market prices.
- `pyxirr`: Financial calculations (XIRR).
- `rich`: Console formatting and tables.
- `sqlite3`: Local data storage.

## Operational Notes
- **Database**: The application automatically looks for `tradebook.db` in the same directory as the source code (`tradebook_app/`).
- **Data File**: A cleaned, deduplicated version of the trades is maintained as `tradebook_app/merged.csv`.
- **Symbol Mapping**: Symbols are expected to be standard exchange codes. The app automatically handles NSE/BSE suffixes for Yahoo Finance.
- **FIFO Accuracy**: Ensure trades are imported in chronological order for the FIFO logic to be most effective, although the code sorts by `trade_date` and `order_execution_time` during retrieval.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saiankit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/saiankit)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
