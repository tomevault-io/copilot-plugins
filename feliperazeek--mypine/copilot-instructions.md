## mypine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of Pine Script trading strategies and a Python backtesting framework for cryptocurrency trading. The repository focuses on pullback-based trading strategies optimized for prop firm requirements (max 10% drawdown, consistent profitability).

## Project Structure

- **`nakinvest/`**: Main directory containing Pine Script strategies and Python backtesting tools
  - **`*.pine`**: Pine Script strategy files (for TradingView)
  - **`backtest/`**: Python backtesting framework with optimization tools
  - **`module2/`**: Additional Pine Script utilities and components
  - **`volatility-strategy/`**: Specialized volatility-based strategies

## Key Components

### Pine Script Strategies (.pine files)

These are TradingView strategies written in Pine Script v5. Main strategies include:

- **EMA Pullback Strategy** (`ema20-pullback.pine`): Core strategy looking for pullback entries to EMA lines with trend filters
- **Prop Firm Strategies** (`prop-firm-strategy.pine`, `prop-firm-optimized.pine`): Optimized for prop firm challenge requirements
- **RSI Divergence** (`rsi-divergences.pine`, `rsi-divergence-pro.pine`): Divergence-based entry systems
- **Cross EMA** (`cross-ema.pine`, `cross-ema-rsi.pine`): EMA crossover strategies
- **ZLSMA Chandelier** (`zlsma-chandelier-strategy.pine`): Advanced moving average with volatility-based exits

### Python Backtesting Framework

Located in `nakinvest/backtest/`, this framework translates Pine Script strategies to Python for server-side backtesting.

**Core Files:**
- `prop_firm_strategy.py`: Main strategy implementation with multi-confluence entry system (RSI divergence, volume confirmation, MA distance filters)
- `run_backtest.py`: Backtest runner with reporting and visualization
- `data_fetcher.py`: Fetches data from Binance, Yahoo Finance, CoinGecko, or CSV files
- `config.yaml`: Strategy configuration file

**Optimization Scripts:**
- `optimize_strategy.py`, `run_optimization.py`: Parameter grid search optimization
- `advanced_optimizer.py`: Advanced optimization with multiple metrics
- `real_data_optimizer.py`: Optimization using real market data
- `test_*.py`: Various testing scripts for different optimization scenarios

## Common Development Commands

### Python Backtesting

```bash
# Setup (first time only)
# Install TA-Lib first (required dependency)
brew install ta-lib  # macOS
# OR
sudo apt-get install ta-lib  # Ubuntu/Debian

# Install Python dependencies
cd nakinvest/backtest
pip install -r requirements.txt

# Run basic backtest
python run_backtest.py

# Run optimization
python run_optimization.py

# Test on multiple coins
python test_all_coins.py

# Download market data
python download_all_data.py
```

### Working with Pine Script Files

Pine Script files are meant to be used in TradingView. To test changes:
1. Edit the `.pine` file locally
2. Copy the contents to TradingView's Pine Editor
3. Apply to chart and run strategy tester

## Architecture & Design Patterns

### Strategy Logic Flow

1. **Data Fetching**: `CryptoDataFetcher` retrieves OHLCV data
2. **Indicator Calculation**: Calculate EMAs, RSI, volume indicators using TA-Lib
3. **Signal Generation**: Multi-confluence system checks:
   - Higher timeframe trend (HTF MA)
   - Pullback depth and proximity to support/resistance
   - RSI divergence or extreme levels
   - Volume confirmation
   - MA distance filter
4. **Risk Management**: Calculate stop loss and take profit based on configuration
5. **Backtesting**: Simulate trades, track equity, calculate metrics

### Configuration System

The Python backtester uses a hierarchical configuration system:
- `config.yaml`: Main configuration file with all parameters
- `strategy_config` dict: Can override config programmatically
- Default values in `PropFirmStrategy.__init__`

### Data Flow

```
Market Data → Indicators → Signals → Risk Checks → Trade Execution → Metrics
                ↓
          (Volume Filter)
          (MA Distance)
          (Session Filter)
```

## Important Development Considerations

### Backtesting Framework

- The Python implementation mirrors the Pine Script logic but uses TA-Lib for indicators
- Commission and slippage are applied to all trades (default: 0.045% commission, 0.01% slippage)
- Prop firm requirements are checked: max 10% drawdown, max 5% daily drawdown
- Reports are generated in HTML format with equity curves and trade history

### Performance Optimization

- Market data is cached in `market_data/` as JSON files
- Optimization results are timestamped and saved as JSON/TXT files
- Grid search can be slow - use narrow parameter ranges for faster results
- The backtest engine processes data vectorially where possible

### Pine Script to Python Translation

When translating Pine Script strategies to Python:
- Pine Script `ta.ema()` → Python `talib.EMA()`
- Pine Script `ta.rsi()` → Python `talib.RSI()`
- Pine Script series indexing `close[1]` → Python `df['close'].iloc[idx-1]`
- Pine Script `strategy.entry()` → Python position tracking and trade recording

### Risk Management Alignment

Both Pine Script and Python implementations should use identical risk parameters:
- Position size: 2% of equity default
- Stop loss: 1% default (can use candle wick or percentage)
- Take profit: 1% default
- Max drawdown: 10% (prop firm requirement)

## File Naming Conventions

- Pine Script optimized strategies: `*-optimized.pine`
- Backtest results: `*_RESULTS_*.json`
- Strategy comparison reports: `*_COMPARISON_*.md`
- Market data: `market_data/SYMBOL_TIMEFRAME_PERIOD.json`

## Data Sources

The backtester supports multiple data sources via the `data_source` parameter:
- `binance`: Preferred for crypto (uses CCXT)
- `yahoo`: Alternative source (uses yfinance)
- `csv`: For custom data files

Symbols must match the source format:
- Binance: `BTC/USDT`, `ETH/USDT`
- Yahoo: `BTC-USD`, `ETH-USD`

## Output Files

Running backtests generates:
- `backtest_report.html`: Interactive HTML report with metrics and trade history
- `backtest_chart.png`: Equity curve, drawdown, and P&L distribution charts
- `backtest.log`: Detailed execution logs (if logging enabled)
- `*_RESULTS_*.json`: Machine-readable results for analysis

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/feliperazeek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/feliperazeek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
