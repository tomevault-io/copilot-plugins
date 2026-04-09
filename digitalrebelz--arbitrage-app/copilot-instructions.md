## arbitrage-app

> High-frequency arbitrage trading bot with paper trading capabilities.

# CLAUDE.md - Arbitrage Bot Project Instructions

## Project Context
High-frequency arbitrage trading bot with paper trading capabilities.
Focus: SPEED (milliseconds matter), ACCURACY, RELIABILITY

## Critical Performance Rules
- ALL exchange calls must be async
- Use connection pooling
- Cache orderbooks in memory (max 100ms stale)
- Batch database writes
- Profile code with cProfile for performance issues

## Coding Standards

### Python
- Type hints REQUIRED for all functions
- Docstrings (Google style) for all public functions
- Maximum 80 lines per function
- Async/await for all I/O
- Decimal for all financial calculations (NEVER floats!)

### Naming
- snake_case for functions/variables
- PascalCase for classes
- SCREAMING_SNAKE_CASE for constants
- Prefix async functions with 'async_' if sync variant exists

## Commands
- `make install` - Install dependencies
- `make install-dev` - Install dev dependencies + playwright
- `make test` - Run all tests with coverage
- `make test-fast` - Run unit tests only
- `make test-e2e` - Run E2E tests with Playwright
- `make lint` - Ruff + mypy
- `make format` - Black + isort
- `make run` - Start bot in paper trading mode
- `make dashboard` - Start Streamlit dashboard

## Exchange API Keys
- NEVER put API keys in code
- Use .env file (not in git!)
- Paper trading works without keys

## Architecture

### Directory Structure
```
src/
├── config/         # Settings, constants, logging
├── models/         # Pydantic models
├── exchanges/      # Exchange connectors (async)
├── arbitrage/      # Detection and calculation
├── execution/      # Paper trading, validation
├── risk/           # Risk management
├── database/       # SQLAlchemy models, CRUD
└── ui/             # Streamlit dashboard
```

### Key Components
- `ArbitrageDetector`: Scans for opportunities across exchanges
- `ArbitrageCalculator`: Computes profits with fees and slippage
- `PaperTrader`: Simulates trade execution realistically
- `OrderValidator`: Checks if orders would have executed
- `SlippageSimulator`: Estimates slippage based on orderbook

## Testing
- Unit tests: `tests/unit/`
- Integration tests: `tests/integration/`
- E2E tests: `tests/e2e/` (Playwright)
- Screenshots: `tests/screenshots/e2e/`
- Target coverage: >80%

## Financial Calculations
- ALWAYS use `Decimal` for money (never float!)
- Track fees in both percentages and absolute values
- Estimate slippage based on orderbook depth
- Validate "would-have-executed" against actual orderbook

## Error Handling
- Log all errors with context
- Retry transient failures (network, rate limits)
- Never crash on single trade failure
- Track error rates for monitoring

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digitalrebelz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/digitalrebelz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
