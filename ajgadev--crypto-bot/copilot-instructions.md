## crypto-bot

> Automated crypto spot trading bot for Binance. Runs via cron every 15 minutes on a Hetzner VPS. Three strategies: Mean Reversion (MR), Trend Follow (TF), and Momentum (MOM). Uses 1h candles for indicators. Regime-adaptive MR parameters auto-switch between bull/bear market configs.

# Crypto Trading Bot

## Project Overview
Automated crypto spot trading bot for Binance. Runs via cron every 15 minutes on a Hetzner VPS. Three strategies: Mean Reversion (MR), Trend Follow (TF), and Momentum (MOM). Uses 1h candles for indicators. Regime-adaptive MR parameters auto-switch between bull/bear market configs.

## Tech Stack
- Python 3.12+, async (asyncio + httpx)
- pydantic-settings for config (loads from `.env`)
- SQLite for state (trades, idempotency keys, KV store)
- Telegram for notifications + AI daily reports (Claude Haiku)
- Tests: pytest + pytest-asyncio
- Deployment: rsync to VPS + GitHub Actions CI/CD

## Project Structure
```
src/
  main.py              # Entry point, orchestrates all three strategies
  config/settings.py   # All settings via env vars (pydantic-settings)
  binance/client.py    # Binance API client (httpx async)
  binance/filters.py   # Exchange filters (lot size, notional, etc.)
  binance/types.py     # Data types (Kline, Ticker, etc.)
  strategy/signals.py  # Entry/exit signal logic for MR, TF, and MOM
  strategy/risk.py     # Position sizing
  indicators/          # RSI, EMA, volume SMA, percent change
  execution/
    executor.py        # Order execution (buy/sell) + OCO safety net
    state.py           # SQLite state store (trades, idempotency)
    reconciler.py      # Reconcile local state with exchange
  notifications/telegram.py  # Telegram notifier (trades, reports, open positions)
  reports/daily_ai.py  # AI-powered daily report via Claude Haiku
  backtest/            # Backtesting engine, data loader, report
  logging/json_logger.py
scripts/
  deploy.sh            # Manual rsync deploy to Hetzner VPS
.github/
  workflows/deploy.yml # Auto-deploy on push to main via rsync
tests/                 # pytest tests
data/                  # Backtest CSV data and reports
```

## Key Commands
```bash
# Run bot (dry run)
python -m src.main

# Run tests
pytest

# Lint
ruff check src/ tests/

# Backtest
RUN_MODE=backtest python -m src.main

# Deploy to VPS manually
./scripts/deploy.sh <server-ip>
./scripts/deploy.sh <server-ip> --dry-run
```

## Strategies
- **Mean Reversion (MR)**: Buys on RSI dip + 24h price drop + bullish EMA bias (9/21) + trend filter (EMA 300). Exits on TP/SL/RSI threshold. Places OCO order as safety net after buy.
- **Trend Follow (TF)**: Buys on EMA crossover (20/50) + volume confirmation + RSI range. Exits on trailing stop from peak or death cross. No fixed TP — lets winners run.
- **Momentum (MOM)**: Uses TF entry signals (EMA crossover + volume + RSI) but exits on fixed TP (2.5%) / SL (2.2%). Places OCO order as safety net after buy. Optimized for quick captures of crossover momentum.

## Architecture Notes
- Config: all via env vars, see `src/config/settings.py` for defaults
- Idempotency: keys based on `{strategy}:{symbol}:{side}:{candle_open_ts}` prevent duplicate orders within same candle
- Lock file (`/tmp/trading_bot.lock`) prevents concurrent runs
- State: SQLite DB tracks open/closed trades, highest price (for trailing stop), KV pairs
- Per-strategy symbols: each strategy can have its own symbol list (`MEAN_REVERSION_SYMBOLS`, `TREND_FOLLOW_SYMBOLS`, `MOMENTUM_SYMBOLS`). Empty = falls back to global `SYMBOLS`.
- Strategy order: configurable via `STRATEGY_ORDER` (e.g., `mr,mom,tf`). Controls which strategy gets first pick of the shared budget in FCFS mode, or simply the processing order when budget allocation is enabled.
- Budget allocation: optional per-strategy budget caps (`BUDGET_ALLOCATION_ENABLED`). When enabled, each strategy gets a fixed % of tradable capital (`MR_BUDGET_PCT`, `TF_BUDGET_PCT`, `MOM_BUDGET_PCT`), preventing one strategy from starving others. When disabled, strategies share a single pool (first-come-first-served).
- Defensive mode: optional bear market protection using reference symbol EMA
- Regime-adaptive MR: auto-detects bull/bear via BTC vs EMA200, switches to tighter TP/SL in bear markets
- AI daily report: uses Claude Haiku to generate market analysis sent via Telegram
- Telegram reports include open position details (entry price, current price, unrealized PnL, TP/SL targets for MR and MOM, trailing stop for TF)

## Deployment
- VPS: Hetzner CX23 (Helsinki), runs as cron every 15 min
- Remote dir: `/opt/bots/crypto-bot`
- CI/CD: GitHub Actions auto-deploys on push to main via rsync
- Manual deploy: `./scripts/deploy.sh <server-ip>`

## Run Modes
- `live` — real trades on Binance mainnet
- `dry_run` — simulated trades, real market data
- `backtest` — historical data from CSV files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ajgadev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
