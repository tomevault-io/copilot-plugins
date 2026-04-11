## marietta-gemini

> This file contains project-specific context for the Gemini agent.

# Gemini Project Context: Marietta-Gemini

This file contains project-specific context for the Gemini agent.

## Project Goal

The primary goal of this project appears to be the analysis of cryptocurrency data to develop and test trading strategies. The initial focus has been on setting up a data pipeline to fetch historical crypto data. The presence of academic papers on trend-following strategies suggests this is a key area of interest.

## Technology Stack

- **Language:** Python
- **Core Libraries:**
    - `pandas`: For data manipulation and analysis.
    - `numpy`: For numerical operations.
    - `yfinance`: For fetching financial data from Yahoo Finance.

## Current Status

- **Dependencies:** `requirements.txt` has been created and dependencies have been installed.
- **Data Fetching:** The `src/data_fetcher.py` script has been implemented. It currently fetches historical data for BTC-USD and ETH-USD and saves it to the `data/` directory as CSV files.
- **Data:** The `data/` directory contains `btc-usd.csv` and `eth-usd.csv`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ridermw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ridermw)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
