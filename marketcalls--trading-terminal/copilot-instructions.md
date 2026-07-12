## trading-terminal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MarketCalls Terminal: a single-user, local-first trading terminal (research/charting only, no order execution) with a Zerodha Kite-inspired dark UI. FastAPI backend on port 8000, React (Vite) frontend on port 5173 with `/api` proxied to the backend. Market data comes from yfinance (Yahoo Finance), NSE-focused.

## Commands

```powershell
# Backend (run from backend/; needs a venv with requirements.txt installed)
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000

# Frontend (run from frontend/)
npm run dev      # dev server on 127.0.0.1:5173
npm run build    # tsc -b && vite build - use this as the typecheck
npm run lint     # oxlint
```

There is no test suite; `npm run build` is the frontend correctness gate.

## Architecture

Request path: React Query hooks (`frontend/src/hooks/`) -> axios `/api` (Vite proxy) -> FastAPI routers (`backend/app/routers/`) -> services (`backend/app/services/`) -> yfinance.

**Backend rules**
- ALL yfinance access goes through `services/yfinance_service.py` - never call yfinance from routers. It provides TTL caching (`services/cache.py` decorator; quotes 15s, bars 5min, universe 3min), retry with backoff, symbol normalization (bare `RELIANCE` -> `RELIANCE.NS`), and per-interval range clamping to Yahoo limits (5m/15m max 1mo, 1h max 1y). Cached DataFrames must not be mutated by callers.
- Bar timestamps are tz-naive exchange-local times serialized as unix-seconds-as-if-UTC, so lightweight-charts (which renders UTC) displays NSE session times. Keep this convention for any new series endpoint.
- Indicators (`services/indicators.py`) use TA-Lib except Supertrend and VWAP (manual; VWAP is intraday-only). Every series is pre-shaped for lightweight-charts (`{time, value}` or multi-key records, NaN warm-ups dropped).
- The trend engine is deliberately rule-based and explainable: it returns the `signals` strings that justify the score, which the UI displays. Keep new signals human-readable.
- Movers and breadth share one batched, cached universe download (`get_universe_history`). The universe lives in `app/symbols.py`, is editable, and tolerates dead tickers (e.g. TATAMOTORS.NS was replaced by TMPV.NS/TMCV.NS after the 2025 demerger). Keep `frontend/src/lib/symbols.ts` (search suggestions) in sync with it.
- Route order matters in `routers/analysis.py`: `/trend/watchlist` must be declared before `/trend/{symbol}`.

**Frontend rules**
- Client state (active symbol, timeframe, range, active indicators, search dialog) lives in the zustand store (`store/terminalStore.ts`). All server data goes through React Query with `refetchInterval` polling (quotes 8s, chart/indicators 60s, movers 120s, breadth 300s) and `placeholderData: previous` to avoid skeleton flashes on refetch.
- `ChartPanel` creates the lightweight-charts v5 instance once. Bar data is pushed in one effect; the indicator effect removes and re-adds all indicator series (subpanels get pane index 1+, price pane keeps stretch factor 3). Toggling indicators must never refetch or touch the base candles. `fitContent` runs only when symbol/timeframe/range changes, preserving user zoom on background refreshes.
- shadcn components here are Base UI-based, not Radix: there is no `asChild` prop, and `CommandDialog` children must be wrapped in an explicit `<Command>` root or cmdk crashes.
- Dark-only theme. Kite-style tokens are defined in `src/index.css` (`:root` and `.dark` are identical); custom Tailwind colors: `bull`, `bear`, `warn`, `surface`, `surface-hover`. All prices/percentages/OHLC values use `font-mono`.
- The Vite dev server is pinned to `host: 127.0.0.1` (Chrome resolves localhost to IPv4 while Node binds IPv6 on this machine). TypeScript 6: use `paths` without `baseUrl` (deprecated).

## Gotchas

- yfinance rate limits: batch with `yf.download(list_of_tickers, ...)` instead of looping single tickers; don't poll movers/breadth faster than every 2 minutes.
- lightweight-charts needs unix seconds, ascending, deduplicated - the backend guarantees this in `bars_from_df`; keep it that way.
- `^`-prefixed index symbols must be URL-encoded in paths (the axios helpers already do this).

---
> Source: [marketcalls/trading-terminal](https://github.com/marketcalls/trading-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
