## youwei-stock-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## System Architecture

This is a stock tracking and analysis system with automated data updates. The system follows a data pipeline architecture:

1. **Data Collection** (`stock_data_fetcher.py`): Uses akshare API to fetch real-time stock prices, PE ratios, EPS forecasts, and market cap data
2. **Business Logic** (`stock_calculator.py`): Implements investment calculation logic based on user-defined buy/sell points
3. **Data Processing** (`data_processor.py`): Orchestrates the pipeline, reads Excel config, generates output files
4. **Web Interface** (`streamlit_app.py`): Provides interactive dashboard for data visualization and analysis
5. **Automation** (`.github/workflows/update_stock_data.yml`): GitHub Actions workflow for hourly data updates

## Core Data Flow

The system operates on a configuration-driven model:
- Input: `fromyouwei.xlsx` contains stock tickers and manually set buy/sell points
- Processing: Fetches live data and calculates investment metrics
- Output: `stock_analysis_YYYYMMDD_HHMMSS.csv/json` files consumed by Streamlit

## Key Business Logic

The system implements a specific investment strategy with these key calculations:
- **Mid Target**: (Target Low + Target High) / 2  
- **Potential Upside**: (Mid Target - Current Price) / Current Price × 100%
- **Risk/Reward Ratio**: (Mid Target - Safe Buy High) / (Safe Buy High - Stop Loss)
- **Buy Signals**: Based on price vs Extreme Safe/Safe Buy Low/Safe Buy High thresholds
- **Position Sizing**: Algorithmic recommendations based on current price vs safe zones

## Common Commands

```bash
# Generate/update stock data
python data_processor.py

# Run web application locally  
streamlit run streamlit_app.py

# Install dependencies
pip install -r requirements.txt

# Deploy preparation
chmod +x deploy.sh && ./deploy.sh
```

## Data Sources & APIs

- **akshare.stock_zh_a_spot_em()**: All A-share real-time prices (bulk download, then filter)
- **akshare.stock_profit_forecast_ths()**: EPS forecasts by ticker
- **akshare.stock_individual_info_em()**: Individual stock market cap and info

Important: The system must handle the akshare limitation where real-time prices require downloading ALL A-shares (~5000 stocks) before filtering to target stocks.

## Excel Configuration Schema

The `fromyouwei.xlsx` file drives the system with these required columns:
- Ticker (format: 002156.SZ)
- Name, Exchange, Sector/Theme  
- Safe Buy Low/High, Extreme Safe, Target Low/High, Stop Loss
- Source/Link, Notes, Last Updated

## GitHub Actions Integration

The workflow triggers on:
- Hourly schedule: `cron: '0 */1 * * *'`
- Manual dispatch
- Excel file updates (push to main)

Generated files are auto-committed back to repo, triggering Streamlit redeployment.

## Streamlit Deployment

- Target: Streamlit Community Cloud
- Entry point: `streamlit_app.py`
- Data loading: Reads latest `stock_analysis_*.csv` with 5-minute cache
- Features: Filtering, sorting, signal visualization, data export

## Error Handling Patterns

- API failures: Log warnings, continue processing other stocks
- Missing data: Return None, handle gracefully in calculations  
- Excel format issues: Skip invalid rows, log errors
- File I/O: Use try/catch with meaningful error messages

## Stock Code Format Conversion

Critical: akshare APIs expect different formats:
- Input format: `002156.SZ` (from Excel)  
- akshare format: `002156` (no exchange suffix)
- Conversion: `ticker.split('.')[0]`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WillWeiZ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
