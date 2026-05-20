## project-structure

> This is a Python client library for the Kiwoom REST API, providing easy access to Korean stock market data and trading functionality.

# Kiwoom REST API Project Structure Guide

## Overview
This is a Python client library for the Kiwoom REST API, providing easy access to Korean stock market data and trading functionality.

## Project Structure

### Core Files
- [pyproject.toml](mdc:pyproject.toml) - Project configuration and dependencies
- [README.md](mdc:README.md) - Main documentation and usage examples
- [DEV-README.md](mdc:DEV-README.md) - Development guide

### Source Code Structure
The main source code is located in `src/kiwoom_rest_api/`:

#### Core Modules
- [src/kiwoom_rest_api/core/](mdc:src/kiwoom_rest_api/core/) - Base API classes and utilities
- [src/kiwoom_rest_api/auth/](mdc:src/kiwoom_rest_api/auth/) - Authentication and token management
- [src/kiwoom_rest_api/cli/](mdc:src/kiwoom_rest_api/cli/) - Command-line interface

#### Korean Stock Market Modules
- [src/kiwoom_rest_api/koreanstock/account.py](mdc:src/kiwoom_rest_api/koreanstock/account.py) - Account and trading history APIs
- [src/kiwoom_rest_api/koreanstock/stockinfo.py](mdc:src/kiwoom_rest_api/koreanstock/stockinfo.py) - Stock information and market data
- [src/kiwoom_rest_api/koreanstock/chart.py](mdc:src/kiwoom_rest_api/koreanstock/chart.py) - Chart and technical analysis data
- [src/kiwoom_rest_api/koreanstock/rank_info.py](mdc:src/kiwoom_rest_api/koreanstock/rank_info.py) - Market ranking and statistics
- [src/kiwoom_rest_api/koreanstock/market_condition.py](mdc:src/kiwoom_rest_api/koreanstock/market_condition.py) - Market condition and indicators
- [src/kiwoom_rest_api/koreanstock/elw.py](mdc:src/kiwoom_rest_api/koreanstock/elw.py) - ELW (Equity Linked Warrant) data
- [src/kiwoom_rest_api/koreanstock/sector.py](mdc:src/kiwoom_rest_api/koreanstock/sector.py) - Sector and industry data
- [src/kiwoom_rest_api/koreanstock/foreign_institution.py](mdc:src/kiwoom_rest_api/koreanstock/foreign_institution.py) - Foreign institution trading data
- [src/kiwoom_rest_api/koreanstock/trading.py](mdc:src/kiwoom_rest_api/koreanstock/trading.py) - Trading execution APIs
- [src/kiwoom_rest_api/koreanstock/investor.py](mdc:src/kiwoom_rest_api/koreanstock/investor.py) - Investor behavior data
- [src/kiwoom_rest_api/koreanstock/analysis.py](mdc:src/kiwoom_rest_api/koreanstock/analysis.py) - Market analysis tools

### Test Structure
- [tests/](mdc:tests/) - Test files organized by module
- [tests/koreanstock/](mdc:tests/koreanstock/) - Tests for Korean stock market modules

## Key Dependencies
- `httpx` - HTTP client for API requests
- `typer` - CLI framework
- `rich` - Terminal formatting and output
- `pytest` - Testing framework (dev dependency)

## Development Workflow
1. Use Poetry for dependency management
2. Follow the established API method patterns in existing modules
3. Include comprehensive docstrings with Korean descriptions
4. Add corresponding test cases for new API methods
5. Follow the git commit message format specified in the repository rules

---
> Source: [bamjun/kiwoom-rest-api](https://github.com/bamjun/kiwoom-rest-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
