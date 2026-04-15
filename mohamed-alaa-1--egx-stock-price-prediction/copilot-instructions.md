## egx-stock-price-prediction

> **EGX Stock Price Prediction** is a local, personal-use desktop application for forecasting Egyptian Exchange (EGX) stock prices. It uses deep learning (PyTorch LSTMs with attention) and incorporates quantitative finance principles for risk assessment and signal validation.

# GEMINI.md - Project Context

## Project Overview
**EGX Stock Price Prediction** is a local, personal-use desktop application for forecasting Egyptian Exchange (EGX) stock prices. It uses deep learning (PyTorch LSTMs with attention) and incorporates quantitative finance principles for risk assessment and signal validation.

### Core Technologies
- **UI Framework:** PySide6 (Qt for Python) with Plotly.js for interactive charting.
- **Machine Learning:** PyTorch (LSTM models), supporting both per-stock and Federated Learning (FedAvg) training modes.
- **Data Management:** `yfinance` and `TradingView` providers, Pandas for processing, Parquet for caching, and SQLite for trade journaling.
- **Quant Utilities:** Scipy and Statsmodels for ADF tests, Hurst exponents, VaR (Value at Risk), and Sharpe ratio calculations.

### Architecture
- `src/app/`: Entry point and PySide6 UI components (tabs, panels, widgets).
- `src/ml/`: Model definitions (`lstm_regressor.py`), training loops (`train.py`, `federated_train.py`), and inference logic.
- `src/services/`: Orchestration layer for predictions, training, risk assessment, and strategy execution.
- `src/core/`: Shared schemas, centralized configuration (`config.py`), and quant finance mathematical cores.
- `src/data/`: Data providers, stock universe management, and normalization logic.

## Building and Running

### Prerequisites
- Python 3.11+
- (Optional) TA-Lib for technical indicators (if not on Windows).

### Installation
```bash
pip install -r requirements.txt
```

### Running the Application
```bash
# From project root
python -m src.app.main
```

### Running Tests
```bash
pytest
```

## Development Conventions

### "Local-First" & Safety
- The app is strictly for personal use and educational purposes.
- A "Disclaimer" banner is required in the UI.
- All data and models are stored locally (`data/` directory).

### Constitution Mandates
1. **Statistical Validation:** Every training session must be preceded by signal validation (ADF/Hurst) to ensure data quality.
2. **Risk Companion:** Every ML prediction must be accompanied by risk metrics (VaR 95/99 and Sharpe ratio).
3. **Baseline Comparison:** ML predictions should always be compared against a "Naive" (last close) baseline.

### Coding Style
- **Linting:** Use `ruff` for linting and formatting (config in `ruff.toml` and `pyproject.toml`).
- **Typing:** Use Python type hints extensively.
- **Paths:** Always use `pathlib` for file system operations.
- **Logging:** Use `print` or logger statements with clear prefixes like `[Training]`, `[PredictionService]`, etc.

## Key Files & Directories
- `src/app/main.py`: Main application entry point.
- `src/ml/models/lstm_regressor.py`: LSTM model with optional attention.
- `src/core/config.py`: Global configuration and path management.
- `src/data/egx_stocks.csv`: Current stock universe definitions.
- `data/metadata/trade_journal.sqlite3`: Local database for simulated trades.
- `specs/`: Detailed requirements and implementation tasks for various modules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mohamed-Alaa-1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mohamed-Alaa-1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
