## trading-platform

> - The project directory is project/

# Gemini Added Memories
- The project directory is project/

# Project Overview

This repository provides a robust and modular framework for developing, backtesting, and analyzing quantitative trading strategies. It is designed for quantitative researchers and developers seeking a clean, performant, and extensible platform for their algorithmic trading endeavors.

# Key Directories and Files

-   `configs/`: This directory holds all configuration files for the project. The primary file is `config.yml`.
    -   `config.yml`: A YAML file that centralizes all configurable parameters for backtests and trading strategies. This includes global settings like initial capital, the list of financial instruments (tickers) to be analyzed, and specific parameters for each implemented trading strategy (e.g., window sizes for moving averages, RSI thresholds, stop-loss percentages).

-   `data/`: This directory is designated for storing all historical market data required for backtesting. Data is typically in CSV format.
    -   Data files (e.g., `AAPL.csv`, `MSFT.csv`): Each file contains historical price and volume data for a specific ticker.

-   `scripts/`: Contains standalone Python scripts that orchestrate various parts of the trading platform's workflow.
    -   `download_data.py`: Responsible for fetching historical market data from external sources (e.g., Yahoo Finance via `yfinance`) and saving it into the `data/` directory. It ensures data is in the correct format for the backtesting engine.
    -   `run_backtest.py`: Executes the backtesting process. It reads configurations from `config.yml`, loads the necessary data, initializes the selected trading strategy, and runs the backtest simulation.
    -   `generate_report.py`: Processes the results of one or more backtests to generate comprehensive performance reports, often in markdown or CSV format, summarizing key metrics across strategies and tickers.
    -   `optimize_strategy.py`: Implements optimization algorithms (currently genetic algorithms using DEAP) to find optimal parameters for trading strategies. It iteratively runs backtests with different parameter sets to maximize a defined fitness function (e.g., Sharpe Ratio).

-   `src/`: This is the core source code directory, containing the modular components of the backtesting framework.
    -   `src/backtesting/`:
        -   `engine.py`: The heart of the backtesting system. It simulates trades, manages the portfolio, and calculates equity curves based on market data and strategy signals.
    -   `src/data/`:
        -   `data_loader.py`: Provides utilities for loading historical market data from the `data/` directory into a usable format (e.g., Pandas DataFrames) for the backtesting engine.
    -   `src/portfolio/`:
        -   `manager.py`: Handles all portfolio-related logic, including managing cash, open positions, calculating profit/loss, and recording closed trades.
    -   `src/reporting/`:
        -   `performance.py`: Contains classes and functions for calculating various performance metrics (e.g., Sharpe Ratio, Sortino Ratio, Max Drawdown, CAGR, Win/Loss Ratio) from backtest results.
    -   `src/strategies/`:
        -   `base_strategy.py`: Defines an abstract base class (`BaseStrategy`) that all new trading strategies must inherit from. It outlines the common interface and methods (e.g., `generate_signals`, `_calculate_metrics`) that a strategy must implement.
        -   `moving_average_crossover.py`, `rsi_strategy.py`, `mean_reversion.py`, `lstm_strategy.py`, `pairs_trading_strategy.py`: Concrete implementations of various trading strategies, each extending `BaseStrategy` and defining its unique logic and parameters.
    -   `src/utils/`: A general-purpose directory for utility functions that don't fit into other specific modules.
    -   `src/visualization/`:
        -   `plotter.py`: Contains functions for generating visual representations of backtest results, such as equity curves and drawdowns, typically saving them as image files.

-   `tests/`: This directory contains all unit and integration tests for the codebase.
    -   `test_performance.py`: Example test file for the performance calculation module.

-   `README.md`: The main project documentation, providing a high-level overview, installation instructions, and basic usage examples.

# Configuration Details: `configs/config.yml` Structure

The `config.yml` file is crucial for defining the operational parameters of the backtesting system. Below is an example structure with detailed explanations:

```yaml
# Global backtesting settings
backtest:
  initial_cash: 100000  # Starting capital for the backtest simulation
  tickers:              # List of financial instrument symbols to include in the backtest
    - "AAPL"
    - "MSFT"
    - "GOOGL"

# Configuration for each trading strategy
strategies:
  - moving_average_crossover:  # Parameters for the Moving Average Crossover strategy
      short_window: 50         # Period for the short-term moving average
      long_window: 200         # Period for the long-term moving average
      stop_loss_percentage: 0.02 # Percentage of stop-loss for each trade (e.g., 0.02 = 2%)
  - rsi_strategy:              # Parameters for the RSI Strategy
      rsi_period: 14           # Period for calculating the Relative Strength Index
      overbought_threshold: 70 # RSI value above which an asset is considered overbought
      oversold_threshold: 30   # RSI value below which an asset is considered oversold
      stop_loss_percentage: 0.02
  - mean_reversion:            # Parameters for the Mean Reversion strategy
      window: 20               # Window size for calculating the mean and standard deviation
      num_std_dev: 2           # Number of standard deviations from the mean to trigger trades
      stop_loss_percentage: 0.02
  - lstm_strategy:             # Parameters for the LSTM-based strategy
      look_back: 60            # Number of past data points to consider for LSTM input
      epochs: 10               # Number of training epochs for the LSTM model
      batch_size: 32           # Batch size for LSTM training
      train_split: 0.8         # Proportion of data to use for training the LSTM model
      stop_loss_percentage: 0.02
  - pairs_trading_strategy:    # Parameters for the Pairs Trading strategy
      symbols: ["AAPL", "MSFT"] # The two symbols forming the pair
      window: 60               # Window size for calculating cointegration/spread statistics
      entry_zscore: 2.0        # Z-score threshold to enter a trade
      exit_zscore: 0.5         # Z-score threshold to exit a trade
      stop_loss_percentage: 0.02
```

When adding new strategies or modifying existing ones, ensure their parameters are correctly defined and documented within this `config.yml` structure.

# Strategy Implementation Guidelines

All new trading strategies must adhere to the following guidelines:

1.  **Inheritance:** Each strategy must inherit from `src/strategies/base_strategy.py.BaseStrategy`.
2.  **`__init__` Method:** The constructor should accept `symbol` (for single-asset strategies) or `symbol1`, `symbol2` (for pairs trading) and any strategy-specific parameters (e.g., `short_window`, `rsi_period`). These parameters should be stored as instance variables.
3.  **`generate_signals(self, data)` Method:** This is the core method where the strategy's logic resides. It takes a Pandas DataFrame `data` (containing historical OHLCV data for the relevant symbol(s)) as input and should return a DataFrame with trading signals. Signals should typically be represented as:
    -   `1` for a buy signal
    -   `-1` for a sell signal
    -   `0` for no action
    -   The DataFrame should be indexed by date/time and contain a 'signal' column.
4.  **Internal Calculations:** Any complex calculations (e.g., moving averages, RSI, Z-scores) should be encapsulated within private helper methods (prefixed with `_`) within the strategy class to maintain readability and modularity.
5.  **Docstrings and Type Hints:** All methods and parameters should be thoroughly documented with docstrings and include appropriate type hints for clarity and maintainability.

# Backtesting Workflow

This section outlines the typical steps to run and analyze backtests:

1.  **Download Historical Data:**
    -   **Command:** `python scripts/download_data.py`
    -   **Purpose:** Fetches up-to-date historical price data for the tickers specified in `config.yml` and stores them as CSV files in the `data/` directory. This ensures the backtest uses relevant and current market information.

2.  **Configure Backtest Parameters:**
    -   **Action:** Manually edit `configs/config.yml`.
    -   **Purpose:** Adjust `initial_cash`, `tickers`, and strategy-specific parameters to define the scope and conditions of your backtest. This step is crucial for tailoring the simulation to specific research questions or market scenarios.

3.  **Run Backtest Simulation:**
    -   **Command:** `python scripts/run_backtest.py`
    -   **Purpose:** Executes the backtesting engine using the configured parameters and downloaded data. It simulates trades, manages the virtual portfolio, and generates raw backtest results (equity curve, trade logs). The results are typically saved in the `reports/` directory.

4.  **Generate Performance Report:**
    -   **Command:** `python scripts/generate_report.py`
    -   **Purpose:** Analyzes the raw backtest results to compute various performance metrics (e.g., Sharpe Ratio, Max Drawdown, CAGR). It then compiles these metrics into a human-readable report, often a markdown table, saved in the project's root or `reports/` directory.

5.  **Plot Equity Curves:**
    -   **Command:** `python src/visualization/plotter.py`
    -   **Purpose:** Creates visual representations of the backtest's performance, primarily equity curves, which show the growth of the portfolio over time. These plots are saved as image files (e.g., PNG) in the `reports/` directory, aiding in quick visual assessment of strategy performance.

6.  **Optimize Strategy Parameters:**
    -   **Command:** `python scripts/optimize_strategy.py`
    -   **Purpose:** Initiates a parameter optimization process for a specified strategy. This script uses evolutionary algorithms to search for the best combination of strategy parameters that maximize a given fitness function (e.g., Sharpe Ratio). The output includes the optimal parameters found.

# Testing Procedures

Comprehensive testing is vital for ensuring the reliability and correctness of the trading platform. Tests are organized within the `tests/` directory.

-   **Running All Tests:**
    -   **Command:** `pytest` (from the project root directory)
    -   **Purpose:** Executes all discovered tests within the `tests/` directory.

-   **Running Specific Test Files:**
    -   **Command:** `pytest tests/test_performance.py`
    -   **Purpose:** Runs only the tests defined in `test_performance.py`. This is useful for focused development and debugging.

-   **Running Specific Tests within a File:**
    -   **Command:** `pytest tests/test_performance.py::test_sharpe_ratio_calculation`
    -   **Purpose:** Executes a single test function named `test_sharpe_ratio_calculation` within `test_performance.py`.

-   **Code Coverage (Optional but Recommended):**
    -   **Installation:** `pip install pytest-cov`
    -   **Command:** `pytest --cov=src`
    -   **Purpose:** Generates a report on how much of the `src/` codebase is covered by tests, helping to identify areas that need more testing.

# LLM-Specific Guidelines for Code Modifications

When making any code modifications or additions, please adhere to the following guidelines to maintain code quality, consistency, and facilitate future automated processing:

-   **Code Style (PEP 8):** Strictly follow [PEP 8](https://www.python.org/dev/peps/pep-0008/) for Python code. This includes naming conventions, indentation, line length, and whitespace. Tools like `flake8` or `black` can be used for automated checking and formatting.

-   **Type Hinting:** Utilize [type hints](https://docs.python.org/3/library/typing.html) for all function arguments, return values, and complex variable assignments. This improves code readability, enables static analysis, and reduces bugs.

-   **Docstrings:** Provide comprehensive [docstrings](https://www.python.org/dev/peps/pep-0257/) for all modules, classes, methods, and functions. Docstrings should explain the purpose, arguments, and return values. Use a consistent format (e.g., Google style or reStructuredText).

-   **Modularity and Separation of Concerns:** Maintain the existing modular design. Each module and class should have a single, well-defined responsibility. Avoid creating monolithic functions or classes that handle multiple unrelated tasks. New functionalities should be integrated logically into existing modules or introduced as new, appropriately named modules.

-   **Logging vs. Print Statements:** Use Python's standard `logging` module for any informational, warning, or error output that is not a direct return value of a function. Avoid using `print()` statements for debugging or general information output in core logic, as they can clutter console output and are harder to manage in production environments.

-   **Robust Error Handling:** Implement explicit and robust error handling using `try-except` blocks, especially for operations involving file I/O, network requests, or data processing where unexpected input or external failures might occur. Provide informative error messages.

-   **Dependency Management:** Do not introduce new external libraries without explicit user approval and a clear justification for their necessity. Before suggesting a new library, verify its established usage within the project by checking `requirements.txt` or `setup.py`.

-   **Absolute File Paths:** Always construct absolute file paths using `os.path.join` and `os.path.dirname(__file__)` to ensure portability across different operating systems and execution environments. Avoid hardcoding relative paths.

-   **Optimization Script (`optimize_strategy.py`) Specifics:**
    -   **Suppressing Verbose Output:** When modifying or running `optimize_strategy.py`, ensure that the verbose output from backtest simulations (e.g., detailed buy/sell/stop-loss statements) is suppressed. This is critical for keeping the optimization process clean and focused on the genetic algorithm's generation information. The current implementation achieves this by temporarily redirecting `sys.stdout` to `os.devnull` during backtest runs within the `evaluate_strategy_individual` function. Maintain this behavior.
    -   **Parameter Ranges:** When defining parameter ranges for optimization, ensure they are reasonable and cover a meaningful search space for the respective strategy.
    -   **Fitness Function:** The `evaluate_strategy_individual` function should return a single fitness value (e.g., Sharpe Ratio) that the genetic algorithm aims to maximize.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AryanSarswat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AryanSarswat)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
