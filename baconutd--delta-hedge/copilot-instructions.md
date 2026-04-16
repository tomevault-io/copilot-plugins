## delta-hedge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bitmart is a delta-neutral market making system for cryptocurrency trading. It operates on BitMart exchange for market making and uses Bybit for hedging positions to maintain delta neutrality.

## Development Commands

### Setup and Configuration
```bash
npm run setup              # Complete project setup
npm run setup:init         # Install dependencies
npm run setup:config       # Configure exchange settings
npm run setup:infra        # Set up AWS infrastructure
```

### Development
```bash
npm run dev                # Start development environment
npm run test               # Run test suite
npm run build              # Build for production
npm run deploy             # Deploy to production
```

### Trading System Operations
```bash
npm run market-making      # Start market making engine
npm run risk               # Start risk management system
npm run analytics          # Start analytics platform
npm run data               # Run data pipeline
npm run monitor            # Start system monitoring
```

### Task Management
```bash
npm start                  # Run taskmaster (shows available tasks)
npm run tasks              # List all available tasks
```

## Architecture Overview

The system is designed with the following components:

1. **Market Making Engine** (`src/market-making/`)
   - Manages order placement on BitMart
   - Implements market making strategies

2. **Hedging System** (`src/hedging/`)
   - Maintains delta-neutral positions
   - Executes hedging trades on Bybit

3. **Risk Management** (`src/risk/`)
   - Real-time position monitoring
   - Implements risk limits and emergency stops

4. **Analytics Platform** (`src/analytics/`)
   - Performance tracking
   - P&L calculations

5. **Data Pipeline** (`src/data/`)
   - Market data collection
   - Historical data storage

## Important Configuration

- **taskmaster.json**: Contains all task definitions. Tasks are currently placeholders and need implementation.
- **Environment Variables**: Create `.env` from `.env.example` for API keys and sensitive data
- **Exchange Configuration**: Will be stored in `config/exchanges.json`
- **Risk Parameters**: Will be defined in `config/risk.json`

## Current Status

The project is in initial setup phase. Task automation is configured but implementation is needed for:
- Source code in all modules
- Exchange API integrations
- Risk management logic
- Analytics and monitoring systems

When implementing, ensure all trading logic includes proper error handling and risk checks before executing any trades.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Baconutd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
