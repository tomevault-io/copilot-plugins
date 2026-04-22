## sqlserver-ml-models-nse

> > **Project context file for AI assistants (Claude, Copilot, Cursor).**

# CLAUDE.md — sqlserver_copilot_nse (NSE ML Training Pipeline)

> **Project context file for AI assistants (Claude, Copilot, Cursor).**

---

## 1. SYSTEM OVERVIEW

This is the **NSE ML training pipeline** — one of **7 interconnected repositories** that form an AI-powered stock trading analytics platform. All repos share a single SQL Server database (`stockdata_db`).

### Repository Map

| Layer | Repo | Purpose |
|-------|------|---------|
| Data Ingestion | `stockanalysis` | ETL: yfinance/Alpha Vantage → SQL Server |
| SQL Infrastructure | `sqlserver_mcp` | .NET 8 MCP Server (Microsoft MssqlMcp) — 7 tools (ListTables, DescribeTable, ReadData, CreateTable, DropTable, InsertData, UpdateData) via stdio transport for AI IDE ↔ SQL Server |
| Dashboard | `streamlit-trading-dashboard` | 40+ views, signal tracking, Streamlit UI |
| ML: NASDAQ | `sqlserver_copilot` | Gradient Boosting → `ml_trading_predictions` |
| **ML: NSE** ⭐ | **`sqlserver_copilot_nse`** | **THIS REPO** — 5-model ensemble → `ml_nse_trading_predictions` |
| ML: Forex | `sqlserver_copilot_forex` | XGBoost/LightGBM → `forex_ml_predictions` |
| Agentic AI | `stockdata_agenticai` | 7 CrewAI agents, daily briefing email |

---

## 2. THIS REPO: sqlserver_copilot_nse

### Purpose
Trains a **5-model ensemble** (Random Forest, Gradient Boosting, Extra Trees, Logistic Regression, VotingClassifier) plus **4 regression models** on NSE 500 stocks to predict Buy/Sell signals and price targets. Writes predictions to `ml_nse_trading_predictions`.

### Daily Schedule (Windows Task Scheduler)
```
09:30 AM  Daily prediction run       → ml_nse_trading_predictions
Sunday 2 AM  Weekly full retrain     → Updated model + regressor files
```

### Key Files

```
sqlserver_copilot_nse/
├── src/
│   ├── predict_daily.py             # Daily prediction entry point
│   ├── train_model.py               # Full classifier training pipeline
│   ├── train_regressors.py          # Price regression model training
│   ├── feature_engineering.py       # 90+ feature calculations
│   ├── feature_selection.py         # Feature selection pipeline
│   ├── sql_queries.py               # All SQL queries
│   ├── model_utils.py               # Model save/load
│   ├── ensemble_builder.py          # 5-model ensemble construction
│   └── regressor_builder.py         # 4-regressor ensemble for price targets
├── models/
│   ├── nse_rf_model.pkl             # Random Forest
│   ├── nse_gb_model.pkl             # Gradient Boosting
│   ├── nse_et_model.pkl             # Extra Trees
│   ├── nse_lr_model.pkl             # Logistic Regression
│   ├── nse_voting_model.pkl         # VotingClassifier
│   ├── nse_*_regressor.pkl          # 4 regression models
│   └── feature_columns.pkl          # Selected feature names
├── config/
│   └── settings.py                  # .env configuration
└── logs/
    └── *.log
```

---

## 3. ML MODEL DETAILS

### Classifier Ensemble (5 models)
| Model | Algorithm | Role |
|-------|-----------|------|
| Random Forest | RF Classifier | Ensemble member |
| Gradient Boosting | GB Classifier | Ensemble member |
| Extra Trees | ET Classifier | Ensemble member |
| Logistic Regression | LR | Ensemble member |
| VotingClassifier | Soft voting | Combines all 4 above |

### Regression Models (4)
Used for price prediction targets (complementary to classifier signals).

### Feature Set (90+)
| Category | Examples |
|----------|---------|
| Price-based | Multi-period returns, gap analysis, price ratios |
| Moving Averages | SMA/EMA multiple periods, crossover flags |
| Momentum | RSI, MACD, Stochastic, ROC, CCI, Williams %R, MFI |
| Volatility | Bollinger Bands, ATR, Keltner, historical vol, VIX proxy |
| Volume | Volume ratios, OBV, volume-price trend, accumulation/distribution |
| Fundamental | Market cap category, sector encoding (from fundamentals table) |
| Lag features | Lagged returns and indicators |
| **Market context** | VIX, India VIX, NIFTY 50 return, S&P 500 return, DXY, US 10Y yield, sector NIFTY index return (from `market_context_daily` table) |

### Output Table: `ml_nse_trading_predictions`
| Column | Type | Description |
|--------|------|-------------|
| ticker | VARCHAR | NSE stock symbol |
| trading_date | DATE | Prediction date |
| predicted_signal | VARCHAR | 'Buy' or 'Sell' |
| confidence_percentage | FLOAT | Ensemble confidence (0-100) |
| signal_strength | VARCHAR | 'Strong'/'Moderate'/'Weak' |
| RSI | FLOAT | Current RSI value |
| buy_probability | FLOAT | P(Buy) from ensemble |
| sell_probability | FLOAT | P(Sell) from ensemble |
| model_name | VARCHAR | Model identifier |
| sector | VARCHAR | Stock sector |
| market_cap_category | VARCHAR | Large/Mid/Small cap |
| high_confidence | BIT | Flag for confidence ≥ 70% |

### Also Writes
- `ml_nse_predict_summary` — Daily aggregates + model_accuracy, success_rate_1d/5d/10d
- `ml_nse_technical_indicators` — Technical indicator snapshots

---

## 4. DATABASE CONTEXT

### Tables This Repo READS
| Table | Purpose |
|-------|---------|
| `nse_500_hist_data` | Historical OHLCV (VARCHAR prices — CAST to FLOAT!) |
| `nse_500` | Ticker master list with sector/industry |
| `nse_500_fundamentals` | 37 fundamental metrics per ticker |

### Tables This Repo WRITES
| Table | Purpose |
|-------|---------|
| `ml_nse_trading_predictions` | Daily Buy/Sell predictions with confidence |
| `ml_nse_predict_summary` | Aggregate stats + accuracy tracking |
| `ml_nse_technical_indicators` | Indicator snapshots |

---

## 5. CODING CONVENTIONS

### .env Configuration (Critical)
The `.env` file **must** use the correct SQL Server address matching the NASDAQ repo:
```
SQL_SERVER=192.168.86.28\MSSQLSERVER01
SQL_DATABASE=stockdata_db
SQL_USERNAME=remote_user
SQL_DRIVER=ODBC Driver 17 for SQL Server
SQL_TRUSTED_CONNECTION=no
```
**WARNING**: Do NOT use IP `192.168.87.27` or port-based format (`,1444`). The correct address is `192.168.86.28\MSSQLSERVER01` (named instance). A wrong IP in `.env` caused a 3-day prediction outage in April 2026 that went undetected because the script exited with code 0 on DB failure.

### Critical Data Issues
- **VARCHAR Price Columns**: Same as NASDAQ repo — `CAST(close_price AS FLOAT)` always
- **500 stocks**: Much larger universe than NASDAQ 100. Feature engineering takes longer.
- **Sector/Cap Stratification**: Summary includes model_accuracy by sector and market_cap_category

### Key Differences from NASDAQ Repo
- 5-model ensemble (vs single Gradient Boosting)
- 90+ features (vs 50+)
- Includes regression models for price targets
- Summary table tracks multi-horizon success rates (1d/5d/10d)
- Predictions include sector and market_cap_category columns

---

## 6. DOWNSTREAM CONSUMERS
- **stockdata_agenticai** — ML Analyst + Strategy Trade + Cross-Strategy agents read these predictions
- **streamlit-trading-dashboard** — Displays predictions and tracks accuracy
- `vw_strategy2_trade_opportunities` view joins `ml_nse_trading_predictions` with tech signal data

---

## 7. MCP SERVER FOR DEVELOPMENT

The `sqlserver_mcp` repo provides an MCP server for AI IDEs to query `stockdata_db` directly during development.

### VS Code Configuration
```json
"MSSQL MCP": {
    "type": "stdio",
    "command": "C:\\Users\\sreea\\OneDrive\\Desktop\\sqlserver_mcp\\SQL-AI-samples\\MssqlMcp\\dotnet\\MssqlMcp\\bin\\Debug\\net8.0\\MssqlMcp.exe",
    "env": {
        "CONNECTION_STRING": "Server=192.168.86.28\\MSSQLSERVER01;Database=stockdata_db;User Id=remote_user;Password=YourStrongPassword123!;TrustServerCertificate=True"
    }
}
```

### 7 MCP Tools: ListTables, DescribeTable, ReadData, CreateTable, DropTable, InsertData, UpdateData

Useful for: checking `ml_nse_trading_predictions` output format, verifying `nse_500_hist_data` schema, exploring prediction accuracy and ensemble model results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amirineni111) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
