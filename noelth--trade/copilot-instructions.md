## trade

> - Install Python dependencies: `pip install -r app2/src/requirements.txt`

# CLAUDE.md - Development Guide

## Build & Run Commands
- Install Python dependencies: `pip install -r app2/src/requirements.txt`
- Run backtest with specific strategy: `python app2/src/main.py --ticker AAPL --strategy sma --timeframe 1D`
- Run strategy optimization: `python app2/src/optimization.py --ticker AAPL --strategy SmaCross --optparams "sma_fast_period:20:100,sma_slow_period:50:200"`
- Run API server (port 8765): `cd app2/src && ./run_api.sh`
- Run web dashboard: `cd app2/trade-strategy-dashboard && npm run dev`
- Run tests: `cd app2/src && ./run_tests.sh`

## Testing Commands
- Run all tests: `./run_tests.sh`
- Run API tests only: `./run_tests.sh -t api`
- Run strategy tests only: `./run_tests.sh -t strategy`
- Run candlestick tests only: `./run_tests.sh -t candlestick`
- Run integration tests: `./run_tests.sh -t integration` (requires both servers running)
- Run with verbose output: `./run_tests.sh -v`
- Skip integration tests: `./run_tests.sh -s`

## Code Style Guidelines
- **Python**: Follow PEP 8 guidelines with 4-space indentation
- **Imports**: Group standard library imports first, followed by third-party packages, then local modules
- **Class Naming**: Use CamelCase for strategy classes, snake_case for functions/variables
- **Documentation**: Include docstrings for strategies describing parameters and logic
- **Error Handling**: Use try/except blocks with specific exceptions for API calls
- **Strategy Pattern**: New strategies should inherit from bt.Strategy with params as class attributes
- **Type Hints**: Add type hints to function parameters and return values
- **Parameters**: Support command-line configuration for all strategy parameters
- **Tests**: Write unit tests for new features, ensure proper test coverage

## Project Structure
- **app1**: Simpler trading application (less maintained)
- **app2**: Main trading application with optimization support
  - **src/**: Backend Python code
    - **candlestick_patterns.py**: Candlestick pattern indicators
    - **candlestick_strategy.py**: Strategy using candlestick patterns
    - **api.py**: FastAPI backend API (port 8765)
    - **tests/**: Unit and integration tests
  - **trade-strategy-dashboard/**: Next.js frontend
- **Strategy Development**: Add new strategies to app2/src/strategy.py
- **Pattern Development**: Add new patterns to app2/src/candlestick_patterns.py

## Candlestick Patterns
- All patterns inherit from `CandlestickPatternBase` in candlestick_patterns.py
- Pattern detection logic is implemented in the `next()` method
- Add new patterns to the `CANDLESTICK_PATTERNS` dictionary at the end of the file
- All patterns should have customizable parameters as class attributes
- Write tests for new patterns in tests/test_candlestick_patterns.py

## Testing Guidelines
- Use pytest fixtures for common test data and configurations
- Mock external APIs like Alpaca for unit tests
- Use parameterized tests for checking multiple configurations
- Test both success and error cases
- For new patterns, include tests with data that both triggers and doesn't trigger the pattern

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noelth)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/noelth)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
