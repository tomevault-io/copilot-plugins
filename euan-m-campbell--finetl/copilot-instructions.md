## finetl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FinETL is a Python library for financial data ETL (Extract, Transform, Load) processes. It uses yfinance for financial market data extraction and HuggingFace datasets for data handling.

## Development Commands

```bash
# Install dependencies
poetry install

# Run tests
poetry run pytest

# Add a dependency
poetry add <package>
```

## Architecture

The project follows a standard ETL pattern with four main modules in `src/finetl/`:

- **extraction/** - Data extraction from sources (yfinance for financial data)
- **transformation/** - Data cleaning and processing
- **loading/** - Loading data to destinations
- **pipeline.py** - Orchestrates ETL workflows

## Key Dependencies

- **yfinance** - Financial market data
- **datasets** - HuggingFace datasets library
- **Python 3.12+** required

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Euan-M-Campbell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
