## biasdashboard

> Pre-session trading bias dashboard for Tashkent-based scalper (UTC+5).

# Bias Dashboard — Agent Instructions

## Project overview
Pre-session trading bias dashboard for Tashkent-based scalper (UTC+5).
Fetches live options chain data, computes GEX (Gamma Exposure), full Black-Scholes Greeks,
funding rates, and outputs a single-file HTML dashboard with a 7-level probability scale.

## Single file
All logic lives in `dashboard_v2.py`. Do not split into multiple files.

## Running
```bash
pip install -r requirements.txt

python dashboard_v2.py              # one-shot, saves morning_dashboard.html
python dashboard_v2.py --serve      # live server at http://localhost:8050, auto-refresh every 5 min
python dashboard_v2.py --verify     # prints raw API responses for debugging
python dashboard_v2.py --crypto     # crypto only (skips SPY/GLD/USO)
```

## Data sources (all free, no API keys needed)
| Source | Data |
|--------|------|
| Deribit public API | BTC/ETH options chain → GEX, gamma flip, max pain, P/C ratio |
| Binance public API (`fapi`) | BTC/ETH perpetual funding rates (primary signal) |
| Yahoo Finance (`yfinance`) | SPY / GLD / USO options chain + 5d price history |
| Black-Scholes (local) | Full Greeks (Δ Γ Θ V ρ) computed from IV |

## Key architecture
- `bs_greeks()` — Black-Scholes Greeks calculator
- `deribit_gex(ccy)` — fetches top-80 options by OI, computes GEX per strike
- `binance_funding(ccy)` — Binance perp funding rate (most liquid market)
- `yf_greeks(symbol, hist)` — full Greeks for SPY/GLD/USO via yfinance
- `score(crypto, us_data)` — 7-level conviction scorer (EXTREME BEAR → EXTREME BULL)
- `build_html(...)` — assembles the dashboard HTML
- `fetch_all(serve_mode)` — orchestrates all fetches, sets `_is_fetching` flag
- `/api/status` HTTP endpoint — JSON: `{last_updated, fetching, next_refresh}` — polled by JS every 5s

## Serve mode mechanics
- Background thread calls `fetch_all()` every 300s (measured from end of last fetch)
- `_is_fetching` flag prevents duplicate concurrent fetches
- Browser JS polls `/api/status` every 5s; reloads page only when `last_updated` timestamp advances
- `/refresh` endpoint triggers immediate background re-fetch, returns a spinner page that polls until done

## Data tagging convention
Every number shown in the dashboard is tagged:
- `[LIVE]` = fetched from API
- `[CALC]` = computed via Black-Scholes from fetched IV
- `[FAIL]` = fetch failed — value omitted, never synthesized

## Markets covered
- **Crypto**: BTC, ETH, SOL (Deribit perp + options GEX for BTC/ETH)
- **Equities**: SPY (S&P 500 ETF), GLD (Gold ETF), USO (Crude Oil ETF)

## Coding conventions
- No external dashboard frameworks (Dash, Streamlit, etc.) — pure stdlib HTTP server
- No API keys required — all public endpoints
- `_safe_oi()` / `_safe_float()` used for NaN-safe parsing of yfinance data
- GEX units: USD gamma exposure per 1% move in spot
- All times displayed in UZT (UTC+5)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tim7en)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tim7en)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
