## trading-strategy

> This project is a quantitative trading system designed for automated trading and backtesting across markets (Crypto/Stocks/Futures). It aims to provide modules for data ingestion, strategy development, backtesting, and live execution with risk management.

# Project Context: Quant Trading Project

## Project Overview
This project is a quantitative trading system designed for automated trading and backtesting across markets (Crypto/Stocks/Futures). It aims to provide modules for data ingestion, strategy development, backtesting, and live execution with risk management.

## Technology Stack
*   **Language:** Python 3.9+
*   **Data Ingestion:** python-binance
*   **Data Processing:** Pandas, NumPy
*   **Technical Indicators:** ta (Technical Analysis Library)
*   **Web Dashboard:** Streamlit, Plotly
*   **Environment:** Virtual environment (`venv`)

## Project Structure

```
quant-trading-project/
├── data_ingestion/             # Data Fetching Scripts
│   ├── fetch_history.py        # Fetch historical klines to CSV
│   └── fetch_latest.py         # Fetch latest data for AI analysis
│
├── data/                       # Data Storage
│   └── [SYMBOL]_[INTERVAL]_[DATE].csv
│
├── archive/                    # Archived Modules
│   └── app/                    # Streamlit Dashboard (Deprecated)
│
├── .gemini/                    # AI Agent Configuration
│   └── skills/
│       └── crypto_trading_skill/
│           └── SKILL.md        # AI Trading Strategy Rules
│
├── venv/                       # Python Virtual Environment
├── requirements.txt            # Python Dependencies
└── README.md                   # Project Documentation
```

## Setup & Usage

1.  **Environment Setup:**
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    ```

2.  **Fetch Data:**
    ```bash
    # Historical Data
    python data_ingestion/fetch_history.py
    
    # Latest Analysis Data (for AI)
    python data_ingestion/fetch_latest.py
    ```

## Development History & Status

### 2026-01-21: Pivot to Automation
*   **Archived Web Dashboard**: Moved `app/` to `archive/` to focus on headless automation.
*   **Goal**: Create a Python-based "Auto Bot" that runs 24/7, analyzing the market every hour using the `crypto_trading_skill` logic.

### 2026-01-20/21: Initialization & Prototyping
*   Initialized project with `python-binance` and `pandas`.
*   Created `crypto_trading_skill` (SMC + RSI/MACD/AMD Strategy).
*   Developed `get-suggestion` workflow for creating detailed market reports in Traditional Chinese.
*   Refactored project structure: separated `data_ingestion` scripts.

### Immediate Goals
*   [x] Initialize structure.
*   [x] Prototype strategy analysis (Workflow).
*   [ ] **Develop `auto_bot.py`**:
    *   [ ] Implement logic loops (15m/1h interval checks).
    *   [ ] Translate `SKILL.md` rules into python functions (RSI, MACD, Confidence Score).
    *   [ ] Add console logging/alerts.
*   [ ] Implement Backtesting Engine.
*   [ ] Telegram Notification Bot.

---
> Source: [drakehuang81/trading-strategy](https://github.com/drakehuang81/trading-strategy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
