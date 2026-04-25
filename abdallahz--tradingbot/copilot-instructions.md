## tradingbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

An automated day-trading **alert system** (no trades executed). Scans stocks for intraday setups, sends Telegram notifications, and serves a Flask web dashboard. Deployed on Heroku (web) + Render (crons) with Supabase persistence.

## Commands

```bash
# Install (editable mode)
pip install -r requirements.txt && pip install -e .

# Run locally with mock data (no credentials needed)
python -m tradingbot.cli run-day

# Run with real Alpaca data
python -m tradingbot.cli --real-data run-news       # Night catalyst research
python -m tradingbot.cli --real-data run-morning    # 08:45 ET pre-market scan
python -m tradingbot.cli --real-data run-midday     # 12:00 ET midday scan
python -m tradingbot.cli --real-data run-close      # 15:30 ET close scan

# Show schedule
python -m tradingbot.cli schedule

# Tests (131 tests)
pytest tests/ -v
pytest tests/test_trade_card.py -v   # single file
```

## Architecture

### Three-Option Watchlist System

Every scan session produces three parallel watchlists:
- **Option 1 (Night Research)**: Top 10 catalyst-driven picks from news/SEC/social signals
- **Option 2 (Relaxed)**: Catalyst-weighted ranker (30% catalyst). Separate budget (2 trades/day).
- **Option 3 (Strict)**: Gap ≥0.5%, volume ≥50K, DV ≥$500K, spread ≤2%, score ≥50, max 8 cards

`MarketConditionAnalyzer` recommends which option based on live volatility.

### Data Flow

CLI → `Scheduler` → `SessionRunner` which:
1. Loads catalyst scores (from prior `run-news` job saved to `catalyst_scores.json`)
2. Fetches snapshots via `AlpacaClient`
3. Filters through `GapScanner` (price_min=$5, long-only positive gaps)
4. Detects patterns, checks pullback setups, scores confluence
5. Ranks via `CatalystWeightedRanker` (11 scoring components)
6. `_build_cards` 21-step filter chain (market guard → price guard → inverse ETF blocker → gap fade → catalyst gate → confluence engine → etc.)
7. Builds `TradeCard` (entry/stop/TP1/TP2), blends score 60% ranker + 40% confluence
8. Sends via `TelegramNotifier` (1.5s delay, retries), persists to `AlertStore` (Supabase + JSONL fallback)

### Key Modules

- `src/tradingbot/app/` — Scheduler, session runner (~1200 lines), worker loop (WORKER_ENABLED gate)
- `src/tradingbot/analysis/` — Pattern detection, market guard (SPY/QQQ), market conditions, confluence engine, chart generation
- `src/tradingbot/research/` — News aggregation (SEC EDGAR, RSS, social proxy), catalyst scoring, insider/institutional/congressional tracking
- `src/tradingbot/scanner/` — Gap scanner filters, close/hold scanner
- `src/tradingbot/signals/` — Pullback setup detection (EMA hold, VWAP reclaim, volume confirmation)
- `src/tradingbot/strategy/trade_card.py` — Trade card construction with entry/stop/target placement (MIN_RR=1.5)
- `src/tradingbot/ranking/ranker.py` — 11-component multi-factor scoring (gap 15%, catalyst 15%, relvol 13%, etc.)
- `src/tradingbot/risk/risk_manager.py` — Max 8 trades/day, 3 consecutive loss lockout, streak scaling (75%→50%→lockout)
- `src/tradingbot/data/etf_metadata.py` — ETF family dedup, inverse ETF detection (11 inverse + 2 VIX blocked)
- `src/tradingbot/web/` — Flask dashboard (dark theme), Supabase alert store with trade outcomes
- `src/tradingbot/models.py` — Core dataclasses: `SymbolSnapshot`, `TradeCard`, `ThreeOptionWatchlist`, `RiskState`

### Configuration

All thresholds live in YAML files under `config/`:
- `scanner.yaml` — price_min=$5, min_gap=0.5%, min_score=50, max_candidates=8
- `risk.yaml` — max_trades=8, o2_max=2, stop=2.5%, lockout=1.5%, consecutive=3
- `indicators.yaml` — EMA 9/20, vol spike morning=1.5×, midday=1.3×
- `schedule.yaml` — night 20:00, morning 08:00, premarket 08:45, close 15:30

Environment variables override YAML for cloud deployment. See `config/broker.example.yaml` for credential template.

### Deployment

- **Heroku**: Web dyno only (worker OFF, `WORKER_ENABLED=false`). Flask dashboard + Supabase reads.
- **Render**: 6 cron jobs handle all scheduling (news, morning, midday, tracker, close)
- **Supabase**: Tables: alerts, trade_outcomes, sessions, close_picks

Required env vars: `ALPACA_API_KEY`, `ALPACA_API_SECRET`, `ALPACA_PAPER`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, `SUPABASE_URL`, `SUPABASE_KEY`

## Design Decisions

- **Long-only**: No short setups. `TradeCard` and `CloseHoldPick` have `side: str = "long"` as a constant default.
- **Alert-only**: No order execution — generates trade card recommendations.
- **Free indicators only**: Uses `ta` library (not torch/transformers) to stay within Heroku slug size limits.
- **Telegram-primary**: Telegram is the main notification channel; web dashboard is a secondary alert viewer.
- **Stateless workers**: Heroku/Render dynos can restart anytime — Supabase is the persistent source of truth for alerts.
- **AI optional**: LLM-based sentiment/validation (OpenAI/Anthropic) is opt-in and not required for core functionality.
- **Inverse ETFs blocked**: 11 inverse ETFs + 2 VIX ETFs blocked — going long on inverse = short bet.
- **Secondary price guard**: Hard floor at $5 in `_build_cards` regardless of scanner path.

## Live Deployment

- **Heroku dashboard**: `https://aztradingbot-c8a5462555f3.herokuapp.com`
- **Health check**: `https://aztradingbot-c8a5462555f3.herokuapp.com/api/health`
- **Render config**: `render.yaml` (6 cron jobs — sole scheduler)

## Local Development

```bash
pip install -r requirements.txt && pip install -e .

# Run Flask dashboard locally
FLASK_APP=src/tradingbot/web/app.py python -m flask run --port 5000

# Run scanner with mock data (no credentials needed)
python -m tradingbot.cli run-day
```

## Documentation

- `docs/ALGORITHM.md` — Full trading pipeline, scoring weights, filter chain, business rules
- `docs/OPERATIONS.md` — Deployment, scheduling, file persistence, troubleshooting
- `docs/IMPROVEMENTS.md` — Algorithm improvement tracker with validation verdicts
- `docs/EXECUTION_ENGINE_PLAN.md` — Automated order execution plan (next MVP)

## Known Issues & Next Steps

### Deferred
- **EMA hold check** — uses `pullback_low >= ema20` which may be too loose. Needs analysis.

### Still Open
- **pyproject.toml missing runtime deps** — only lists PyYAML; need to sync from requirements.txt.
- **session_runner.py is ~1200 lines** — should extract CardBuilder and ScanStrategy classes.

### Backlog (biggest potential improvements)
- **Higher-timeframe trend filter** — biggest single edge improvement. Stocks gapping up in a downtrend are bear rallies.
- **Volume decay detection** — track fading participation across bars.
- **Dynamic R:R by score** — high-conviction setups should accept lower R:R.

See `docs/IMPROVEMENTS.md` for the full tracker.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abdallahz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
