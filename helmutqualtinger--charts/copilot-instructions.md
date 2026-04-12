## charts

> Generates year marks every 2 years (2000, 2002, 2004, etc.) for the RangeSlider:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a financial data visualization application that displays index performance comparisons (DAX, S&P 500, SMI, Gold) normalized to a base value of 100 in CHF. The application is built with Flask + Dash and uses Plotly for interactive charts. All data processing (currency conversion, normalization, statistics) is handled server-side in Python.

## Architecture

### Flask + Dash Integration

The application uses **Dash** (built on Flask) as the primary framework:

- **`server.py`** (Main Application):
  - Integrates Dash with Flask for a unified web application
  - Fetches financial data using `yfinance` library
  - Processes data server-side (currency conversion, normalization)
  - Generates interactive Plotly charts dynamically
  - Provides Dash callbacks for reactive UI updates
  - Runs on port 5000

- **`proxy.py`** (Alternative/Legacy): Direct Yahoo Finance API proxy with rate limiting
  - Not used with the Dash implementation
  - Kept for reference

### Application Components

1. **Data Fetching** (server.py:39-79)
   - `fetch_all_data()`: Fetches all ticker data from yfinance
   - Handles: DAX (^GDAXI), SMI (SMIC.SW), S&P 500 (^SP500TR), Gold (GC=F), EUR/CHF, USD/CHF
   - Back-fills exchange rates to ensure consistent date ranges

2. **Data Processing** (server.py:82-144)
   - `process_and_scale_data()`: Converts to CHF, filters to month-end, normalizes to base 100
   - All processing done server-side in Python using pandas
   - Resamples to month-end dates ('ME')
   - Forward-fills missing values
   - Normalizes each index to base 100

3. **Statistics Calculation** (server.py:147-191)
   - `calculate_statistics()`: Computes CAGR and total return for each index
   - Results displayed in the statistics panel

4. **Interactive UI** (server.py:213-259)
   - Dash layout with Plotly chart
   - RangeSlider with year markings (2000, 2002, 2004, etc.)
   - Real-time date labels showing selected range
   - Performance statistics panel
   - Dark theme for better readability

5. **Callbacks** (server.py:272-358)
   - Date label update callback (updates "Von:" and "Bis:" labels)
   - Clientside callback for tooltip formatting (converts timestamps to dates)
   - Main chart update callback (triggers on slider change)

### Data Flow

1. User opens `http://localhost:5000`
2. Dash loads the application layout automatically
3. On page load, the slider triggers data fetching
4. Backend fetches data from yfinance for all tickers
5. Backend converts foreign currency indices to CHF
6. Backend filters to month-end dates and normalizes to base 100
7. Plotly renders the interactive performance comparison chart
8.  When user adjusts slider, chart automatically updates

## Development Commands

### Running the Application

Start the Dash server:
```bash
python3 server.py
```

The application will be available at `http://localhost:5000`

### Dependencies

Install dependencies using pip:
```bash
pip install -r requirements.txt
```

Or using uv (recommended):
```bash
uv pip install -r requirements.txt
```

Required packages:
- Flask >= 3.0.0
- yfinance >= 0.2.0
- pandas >= 2.0.0
- dash >= 2.14.0
- plotly >= 5.18.0

## Key Implementation Details

### Ticker Mappings (server.py:21-28)

```python
TICKERS = {
    'dax': '^GDAXI',       # DAX Total Return
    'smi': 'SMIC.SW',      # SMI Total Return Index
    'sp500': '^SP500TR',   # S&P 500 Total Return
    'gold': 'GC=F',        # Gold Futures
    'eurChf': 'EURCHF=X',
    'usdChf': 'USDCHF=X'
}
```

### Index Configuration (server.py:31-36)

```python
INDEXES = [
    {'name': 'DAX (TR)', 'ticker': 'dax', 'currency': 'EUR', 'color': 'rgb(0, 104, 182)'},
    {'name': 'S&P 500 (TR)', 'ticker': 'sp500', 'currency': 'USD', 'color': 'rgb(75, 192, 192)'},
    {'name': 'SMI (TR)', 'ticker': 'smi', 'currency': 'CHF', 'color': 'rgb(255, 99, 132)'},
    {'name': 'Gold', 'ticker': 'gold', 'currency': 'USD', 'color': 'rgb(255, 215, 0)'}
]
```

### Data Normalization Logic

Server-side processing (server.py:82-144):
1. Converts EUR/USD indices to CHF using exchange rates
2. Filters by selected date range
3. Resamples to month-end dates only (last date of each month)
4. Forward-fills missing values
5. Normalizes each index to 100 at first valid value

### Slider Marks Generation (server.py:194-209)

Generates year marks every 2 years (2000, 2002, 2004, etc.) for the RangeSlider:
```python
def generate_slider_marks():
    marks = {}
    for year in range(2000, end_year + 1, 2):
        timestamp = datetime(year, 1, 1).timestamp()
        marks[timestamp] = {'label': str(year), 'style': {'color': '#e0e0e0'}}
    return marks
```

### Clientside Callback for Tooltips (server.py:284-314)

Uses JavaScript to format slider tooltips from Unix timestamps to human-readable dates (YYYY-MM-DD).

## Common Modifications

### Adding a New Index

1. Add ticker mapping in `server.py` TICKERS dict (line 21-28)
2. Add configuration in `server.py` INDEXES list (line 31-36)
3. Specify currency and color
4. Exchange rate ticker will be used automatically if currency is EUR or USD

### Changing Date Range

Default date range: 2000-01-01 to present (server.py:43)

To modify, change the `start_date` in `fetch_all_data()` function.

### Adjusting Chart Display

Chart configuration is in the `update_chart()` callback (server.py:287-332). Modify Plotly's `fig.update_layout()` for appearance, colors, axes, etc.

### Customizing Slider

- Modify `generate_slider_marks()` to change year intervals (server.py:194-209)
- Adjust slider styling in the layout (server.py:238-246)
- Tooltip formatting is handled by the clientside callback (server.py:284-314)

## Notes

- All data processing is now done server-side (Python/pandas) instead of client-side (JavaScript)
- Dash handles the UI reactivity through callbacks
- Plotly provides interactive charts with zoom, pan, and hover capabilities
- The application uses dark theme for better readability (#1a1a1a background)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HelmutQualtinger)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HelmutQualtinger)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
