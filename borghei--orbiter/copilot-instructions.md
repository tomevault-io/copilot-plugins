## orbiter

> uv venv && source .venv/bin/activate && uv pip install -e ".[dev,dashboard,ai]"

# Orbiter Development Guide

## Setup
uv venv && source .venv/bin/activate && uv pip install -e ".[dev,dashboard,ai]"

## Commands
- Run tests: pytest
- Run single test: pytest tests/test_metrics.py::test_sharpe_ratio -v
- Lint: ruff check src/ tests/
- Format: ruff format src/ tests/
- CLI: orbiter optimize BTC ETH SOL --strategy max-sharpe
- Dashboard: streamlit run src/orbiter/dashboard.py

## Architecture
- src/orbiter/data.py — price fetching via ccxt
- src/orbiter/metrics.py — pure functions, no side effects
- src/orbiter/covariance.py — covariance matrix estimators
- src/orbiter/optimize.py — PortfolioOptimizer class, 10 strategies
- src/orbiter/backtest.py — walk-forward engine
- src/orbiter/black_litterman.py — Black-Litterman model + AI view parsing
- src/orbiter/ai.py — AI middleware (Claude, OpenAI, Perplexity adapters)
- src/orbiter/prompts.py — prompt templates for AI analysis
- src/orbiter/sentiment.py — Fear & Greed, funding rates, exchange flows
- src/orbiter/defi.py — DeFi yield data from DeFiLlama
- src/orbiter/regime.py — HMM regime detection + sentiment-enhanced variant
- src/orbiter/cli.py — Click CLI
- src/orbiter/dashboard.py — Streamlit app

## Conventions
- Log returns throughout (not simple returns)
- Type hints on all public functions
- Docstrings: Google style, only where logic isn't obvious
- All optimization assumes long-only (no short selling)
- Default periods_per_year=365 (crypto trades 24/7)
- Default covariance method: ledoit-wolf
- AI SDKs imported lazily (inside methods) to avoid hard dependencies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/borghei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
