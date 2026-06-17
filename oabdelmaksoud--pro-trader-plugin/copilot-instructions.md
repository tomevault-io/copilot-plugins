## pro-trader-plugin

> ProTrader is a multi-agent trading skill that scans markets, debates trade setups, and executes trades through Alpaca with full risk management. It combines technical analysis, macro fundamentals, sentiment analysis, and guru tracking into a single autonomous pipeline.

# ProTrader — Autonomous Multi-Agent Trading Skill

## Description

ProTrader is a multi-agent trading skill that scans markets, debates trade setups, and executes trades through Alpaca with full risk management. It combines technical analysis, macro fundamentals, sentiment analysis, and guru tracking into a single autonomous pipeline.

## What This Skill Does

1. **Market Scanning** — Scans tier-1 and tier-2 watchlists across 7 parallel AI agents (technical, macro, sentiment)
2. **Trade Debate** — Runs Bull vs Bear debate rounds before any entry, judged by Claude Opus
3. **Risk Gating** — Every trade passes through 9 checkpoints: score threshold, conviction floor, drawdown check, correlation limit, VIX sizing, Kelly sizing, max positions, daily loss cap, and circuit breaker
4. **News Monitoring** — Scans 20 RSS feeds + Finnhub every 2 minutes for breaking catalysts
5. **Guru Tracking** — Monitors 10 hedge fund managers via 13F filings + STOCK Act disclosures for alpha signals
6. **Post-Trade Learning** — Reflects on every closed trade, stores lessons in BM25 memory for future decisions

## Usage Examples

### Run a full market scan
```
Scan the market for the best trading setups right now. Use the ProTrader pipeline with all agents.
```

### Analyze a specific ticker
```
Run the full ProTrader analysis on NVDA — technical, macro, sentiment, and debate. Tell me if it's a buy.
```

### Check portfolio risk
```
Check my current drawdown state, circuit breaker status, and open position risk using ProTrader.
```

### Monitor breaking news
```
Run the ProTrader breaking news scanner and classify any stories by tier (war/Fed = tier 1, earnings = tier 2, analyst = tier 3).
```

### Review trade performance
```
Run a ProTrader post-trade reflection on my last closed position. Store the lesson in memory.
```

### Run a backtest
```
Backtest the ProTrader strategy on AAPL for the last 3 months. Show win rate, avg win, avg loss, and Sharpe.
```

## Configuration

The skill reads from `config/strategy.json` for:
- **Scoring weights**: catalyst (0.3), technical (0.25), sentiment (0.2), fundamental (0.15), risk/reward (0.1)
- **Entry thresholds**: score >= 7.0, conviction >= 7 (tighter in last trading window)
- **Position sizing**: Kelly-based with conviction scaling, max 2 concurrent positions
- **Risk limits**: 5% max daily loss, VIX-adaptive sizing, sector correlation caps
- **Watchlist**: 13 tier-1 tickers, 6 sector-based tier-2 groups, 10 ETFs

## Required Environment Variables

```
ALPACA_API_KEY=       # Alpaca paper/live trading API key
ALPACA_SECRET_KEY=    # Alpaca secret key
FINNHUB_API_KEY=      # Finnhub news API key (optional, enhances news scanning)
```

## Key Files

| File | Purpose |
|---|---|
| `config/strategy.json` | All strategy parameters — thresholds, sizing, risk limits |
| `scripts/breaking_news_monitor.py` | Standalone news scanner (runs every 2 min) |
| `scripts/guru_tracker.py` | Hedge fund + politician trade tracker |
| `scripts/drawdown_monitor.py` | Circuit breaker — halts trading on 5%+ daily drawdown |
| `scripts/reflect_on_trade.py` | Post-trade LLM reflection → BM25 memory |
| `logs/situation_memory.json` | BM25 persistent memory (max 500 entries) |
| `logs/kelly_params.json` | Rolling Kelly criterion parameters |
| `logs/drawdown_state.json` | Current drawdown and halt state |

## Notes

- All inference routes through OpenClaw's native model routing — no separate LLM API keys needed
- The system is designed for paper trading via Alpaca but can be pointed at live endpoints
- News monitor runs as a standalone process to avoid context overflow (previous inline approach crashed at 191K tokens)
- Strategy parameters in `config/strategy.json` should be adjusted to match your account size

---
> Source: [oabdelmaksoud/Pro-Trader-PLUGIN](https://github.com/oabdelmaksoud/Pro-Trader-PLUGIN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
