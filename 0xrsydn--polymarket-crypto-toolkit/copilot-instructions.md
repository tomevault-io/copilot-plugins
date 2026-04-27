## polymarket-crypto-toolkit

> Composable Python toolkit for backtesting and live execution on Polymarket prediction markets. Plugin-based strategies and indicators, multi-source data feeds.

# AGENTS.md — Polymarket Trading Toolkit

## Project
Composable Python toolkit for backtesting and live execution on Polymarket prediction markets. Plugin-based strategies and indicators, multi-source data feeds.

## Stack
- Python 3.13, uv workspaces (monorepo)
- Nix flake devshell (`nix develop`) — optional, uv-only works too
- py-clob-client, web3, websockets, pandas, numpy
- Ruff (lint), ty (typecheck), pytest, prek (git hooks)

## Structure
```
packages/
  core/         → Protocol types (Strategy, Indicator, DataFeed, PriceTick), config, plugin registry
  data/         → Binance OHLCV fetcher + storage (CSV/Parquet)
  indicators/   → EMA, SMA, RSI, MACD, Bollinger Bands
  strategies/   → Streak reversal, copytrade, candle direction, selective filter
  backtest/     → Engine + parameter sweep + walk-forward + metrics
  executor/     → Polymarket CLOB client, WebSocket, trader, blockchain, resilience
scripts/        → CLI entry points
examples/       → Custom strategy plugin example
docs/           → Architecture, conventions, decisions
```

## Development
```bash
# Nix users
nix develop                     # auto: uv sync, prek install

# Non-Nix users
uv sync --all-packages          # install all workspace packages
prek install                    # install git hooks

# Run
uv run python scripts/bot.py --paper
uv run python scripts/backtest.py
uv run pytest -v
```

## Docs
- `docs/ARCHITECTURE.md` — system design, package layering, data flow, DataFeed protocol
- `docs/CONVENTIONS.md` — code patterns, naming, module boundaries
- `docs/DECISIONS.md` — architecture decision records

## Verification
```bash
ruff check packages/ tests/     # Lint
ruff format --check packages/   # Format
ty check                        # Typecheck
uv run pytest -v                # Tests
```

## Rules
- Paper trade first (`--paper`). Never default to live.
- All config via `.env` — no hardcoded keys or amounts.
- New strategies must conform to `Strategy` Protocol (see `packages/core/`).
- New data feeds must conform to `DataFeed` Protocol.
- REST fallback for every WebSocket path (graceful degradation).

---
> Source: [0xrsydn/polymarket-crypto-toolkit](https://github.com/0xrsydn/polymarket-crypto-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
