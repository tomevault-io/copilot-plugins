## arbitrage

> A production trading bot that exploits the latency gap between Binance/Bybit spot prices and Polymarket CLOB prediction markets on 5-minute and 15-minute BTC/ETH/SOL Up/Down micro-buckets. Also supports cross-platform arbitrage against Drift BET on Solana.

# Polymarket Crypto Micro-Arb Bot

## What This Is

A production trading bot that exploits the latency gap between Binance/Bybit spot prices and Polymarket CLOB prediction markets on 5-minute and 15-minute BTC/ETH/SOL Up/Down micro-buckets. Also supports cross-platform arbitrage against Drift BET on Solana.

**DO NOT** place real orders, modify risk parameters, or switch to live mode without explicit user approval.

## Architecture

```
polymarket_micro_arb/
├── data/
│   ├── gamma_client.py        # Polymarket market discovery (Gamma API)
│   ├── binance_ws.py          # Binance + Bybit WS (multi-venue prices)
│   ├── polymarket_ws.py       # CLOB book data WS
│   └── drift_client.py        # Drift BET market discovery + price polling
├── strategy/
│   ├── momentum_latency.py    # Core: 15-45s window, ≥0.35% move, volume+venue confirm
│   ├── cross_outcome_arb.py   # YES+NO < $0.99 same-platform arb
│   └── cross_platform_arb.py  # Polymarket vs Drift price discrepancy arb
├── execution/
│   ├── clob_executor.py       # Polymarket CLOB orders (EIP-712 signing)
│   └── drift_executor.py      # Drift BET orders (Solana keypair)
├── risk/
│   └── risk_engine.py         # Kelly sizing, drawdown kill-switch, position limits
├── dashboard/
│   └── state.py               # JSON state writer for Streamlit
├── utils/
│   ├── logger.py              # Structured JSON logging (loguru)
│   ├── telegram_alerts.py     # Trade/risk/daily Telegram alerts
│   └── backtester.py          # Historical kline replay with full risk engine
├── config.py                  # Pydantic settings from .env
├── models.py                  # Data models (MarketInfo, DriftMarket, Signal, etc.)
├── constants.py               # Enums, timing constants, symbol mappings
└── main.py                    # Bot orchestrator (asyncio.TaskGroup, 10 tasks)
```

## Three Strategies

1. **Momentum Latency** (`strategy/momentum_latency.py`): Binance/Bybit move → Polymarket hasn't repriced → buy underpriced token. 15-45s window, ≥0.35% threshold, volume + multi-venue confirmation.

2. **Cross-Outcome Arb** (`strategy/cross_outcome_arb.py`): YES ask + NO ask < $0.99 on Polymarket → buy both → guaranteed profit.

3. **Cross-Platform Arb** (`strategy/cross_platform_arb.py`): Same event priced differently on Polymarket vs Drift BET → buy the cheaper side, or lock profit by buying opposite sides across platforms. Requires `DRIFT_ENABLED=true` in `.env`.

## Key Files

- `main.py` — Entry point. Launches 10 concurrent tasks via TaskGroup.
- `config.py` — All settings from `.env`. Import `settings` singleton.
- `strategy/momentum_latency.py` — Core strategy. Only fires in 15-45s window.
- `strategy/cross_platform_arb.py` — Compares Polymarket vs Drift prices.
- `risk/risk_engine.py` — Non-negotiable: 4% max/trade, -8% kill-switch, 3 consec loss cooldown, 8 max buckets.
- `execution/clob_executor.py` — Polymarket limit orders, 60s TTL auto-cancel.
- `execution/drift_executor.py` — Drift BET orders (live or paper).
- `dashboard.py` — Streamlit UI at port 8501.

## Running

```bash
# Paper trade (default)
python -m polymarket_micro_arb.main

# Dashboard (separate terminal)
streamlit run dashboard.py --server.port 8501 --server.address 0.0.0.0

# Health check
python scripts/health_check.py

# Daily report
python scripts/daily_report.py
```

## Trading Modes

| Mode | .env Value | Description |
|------|-----------|-------------|
| Paper Trade | `paper_trade` | Real data feeds, simulated fills. Default. |
| Live | `live` | Real CLOB orders with real USDC. Requires funded wallet. |
| Backtest | `backtest` | Replays CSV kline data from `data/backtest/`. |

## Risk Rules (never weaken these)

- Max 4% bankroll per trade (Kelly-sized, hard-capped)
- Daily loss -8% → 24h full pause
- 3 consecutive losses → 30min cooldown (auto-resumes)
- Max 8 concurrent bucket positions
- Limit orders only, auto-cancel unfilled after 60s
- Low balance alerts via Telegram

## Environment

- Python 3.12.3, uv-managed venv at `.venv/` (use `uv pip`, not `pip`)
- Key deps: `py-clob-client`, `websockets` v16, `aiohttp`, `pydantic`, `loguru`, `streamlit`
- Config via `.env` (see `.env.example`)
- Secrets: `PRIVATE_KEY` (Polygon EOA), `SOLANA_PRIVATE_KEY` — NEVER log or display these
- Polygon chain (ID 137), wallet funded with MATIC + USDC.e
- websockets v16: `is_connected` uses `ws.state == 1` (not `ws.open` which was removed)

## Systemd Services

Both run as user services, auto-restart, survive reboots:
```bash
systemctl --user status polymarket-bot
systemctl --user status polymarket-dashboard
systemctl --user restart polymarket-bot
journalctl --user -u polymarket-bot -f        # live logs
```

## Drift BET Integration

Enable with `DRIFT_ENABLED=true` in `.env`. Requires:
- `SOLANA_PRIVATE_KEY` — Solana keypair (base58) for live Drift trading
- `DRIFT_GATEWAY_URL` — Self-hosted Drift Gateway for live orders (default `http://localhost:8080`)
- `CROSS_PLATFORM_MIN_SPREAD` — Minimum spread to trigger cross-platform arb (default 6%)

**Architecture:**
- Data reads use Drift's public APIs (no auth needed):
  - Market discovery: `https://data.api.drift.trade/stats/markets`
  - L2 orderbook: `https://dlob.drift.trade/l2?marketIndex=N&marketType=perp`
- Order execution uses Drift Gateway (self-hosted Rust binary/Docker):
  ```bash
  docker run -e DRIFT_GATEWAY_KEY=<base58_seed> -p 8080:8080 \
    ghcr.io/drift-labs/gateway https://api.mainnet-beta.solana.com
  ```
- BET markets are perp markets with `-BET` suffix. Long=YES, Short=NO.
- Paper trade mode works without Gateway — simulates fills locally.

## Standalone Predictor (`predictor/`)

XGBoost-based crypto price direction predictor. Fully standalone from the arb bot.

```bash
python -m predictor fetch              # Download Binance klines
python -m predictor train              # Walk-forward XGBoost training
python -m predictor predict --once     # Single prediction pass
python -m predictor backtest           # Evaluate on historical data
streamlit run predictor_dashboard.py   # Prediction dashboard
```

## Standalone Predictor (`predictor/`)

XGBoost-based crypto price direction predictor. Fully standalone from the arb bot.

```bash
python -m predictor fetch              # Download Binance klines
python -m predictor train              # Walk-forward XGBoost training
python -m predictor predict --once     # Single prediction pass
python -m predictor backtest           # Evaluate on historical data
streamlit run predictor_dashboard.py   # Prediction dashboard
```

## Common Tasks

**Check if bot is healthy:** `python scripts/health_check.py`
**View live dashboard:** `http://server-ip:8501`
**Check logs:** `ls logs/` — JSON structured, rotated at 50MB
**Generate daily report:** `python scripts/daily_report.py`
**Force unpause risk engine:** Not available via script — restart the bot

## When Making Changes

- Always test in `paper_trade` mode first
- Never weaken risk parameters without explicit user approval
- The strategy's 15-45s window and 0.35% threshold are tuned for high winrate — don't lower them
- All WebSocket clients auto-reconnect; don't add manual restart logic
- State file `bot_state.json` is the bridge between bot and dashboard — don't change its schema without updating both sides
- Drift integration is opt-in (`DRIFT_ENABLED`) — don't make it required
- Commit messages should describe what changed AND why

## Monitoring Agents

- **Health check** (`scripts/health_check.py`): Run every 5 min via `/loop`. Checks process, state freshness, connections, risk status, log errors. Exit code 0=healthy, 1=warning, 2=critical.
- **Daily report** (`scripts/daily_report.py`): Run daily via `/schedule`. Generates full P&L summary with ASCII equity curve, saves to `reports/`, sends Telegram digest.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/charleytrades)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/charleytrades)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
