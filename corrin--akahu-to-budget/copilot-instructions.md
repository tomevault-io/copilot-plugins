## akahu-to-budget

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python-based transaction sync application that bridges Akahu (NZ bank aggregator) with budget management tools (YNAB and Actual Budget). The application syncs bank transactions from Akahu to either or both budgeting platforms.  It's designed to run unmonitored, so if it identifies an issue then it should create quality logs and proactively contact the owner - hiding errors would risk financial disaster.

While it techncially supports a web front end (Flask), no users currently use this.  It's more a legacy coincidence.

## Architecture

- **Sync Entry Point**: `sync_cli.py` - CLI sync functionality without Flask dependencies
- **Web Entry Point**: `flask_app.py` - Flask web application with webhook endpoints
- **Core Modules** (in `/modules/`):
  - `config.py` - Environment variable handling and API configuration
  - `sync_handler.py` - Core sync logic for both YNAB and Actual Budget
  - `account_mapper.py` - Maps Akahu accounts to budget application accounts
  - `transaction_handler.py` - Transaction processing and formatting
  - `webhook_handler.py` - Flask webhook endpoints for real-time sync
  - `account_fetcher.py` - Fetches account data from APIs
- **Setup Script**: `akahu_budget_mapping.py` - Interactive account mapping setup
- **Configuration**: Account mappings stored in `akahu_budget_mapping.json`

## Common Development Commands

### Environment Setup
```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate  # Linux/Mac
# or
.\.venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt

# Install development dependencies (optional)
pip install -r requirements_dev.txt
```

### Running the Application
```bash
# One-time sync (recommended for most users)
python sync_cli.py

# Run webhook server (includes initial sync)
python flask_app.py
```

### Setup Commands
```bash
# Interactive account mapping setup (run once)
python akahu_budget_mapping.py
```

### Development and Testing
```bash
# Lint code
flake8 .

# Run validation tests
python test_transactions.py

# Debug specific transactions
python search_transaction.py
```

## Configuration Requirements

The application requires a `.env` file with API credentials and feature flags. See `.env.example` for the complete template. Key variables:

- `RUN_SYNC_TO_YNAB` and `RUN_SYNC_TO_AB` - Boolean flags to enable/disable sync targets
- `FORCE_REFRESH` - Forces deletion of local Actual Budget cache
- `DEBUG_SYNC` - Enables additional sync-related logging

## Important Notes

- Uses `actualpy` library with context manager pattern for Actual Budget connections
- Supports syncing to both YNAB and Actual Budget simultaneously
- Account mapping configuration is crucial - run `akahu_budget_mapping.py` before first sync
- All logging goes to both `app.log` and console
- Webhook endpoints provide real-time transaction syncing capabilities

## Development Guidelines

- **DO NOT** access SQLite database directly - always use `actualpy` library
- **DO NOT** use placeholders

ALWAYS
- Fail early, and loudly
- Remember DRY
- Follow python best practice
- write code ready for release to production on many different user's computers

---
> Source: [corrin/akahu_to_budget](https://github.com/corrin/akahu_to_budget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
