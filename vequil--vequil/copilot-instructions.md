## vequil

> > This file is the shared source of truth for all agents working on this codebase.

# Vequil — Team Context

> This file is the shared source of truth for all agents working on this codebase.
> Read this at the start of every session before making any changes.
>
> **Claude Terminal (Claude Code in the terminal):** This is auto-loaded on session start.
> **Antigravity (Claude in the IDE):** Read this with view_file at session start, or ask the user to paste it.

---

## Who We Are

- **Noah** — founder/operator. Gives direction, approves plans, owns the product vision.
- **Antigravity** — Primary agent in the IDE. Full access to the filesystem and terminal. Handles high-level architecture, complex multi-file refactors, and can execute bash commands directly to verify builds and run tests.
- **Claude Terminal** — Claude Code running in the terminal (`claude` command). Handles bash execution, API probing, dependency installation, and file writes directed by Antigravity's plans.

**Coordination rule:** Antigravity designs and plans. Claude Terminal executes. Both agents write to this file when something material changes.

---

## What Vequil Is

A **multi-agent prediction market trading framework** and internal trading operation targeting Kalshi — the CFTC-regulated US prediction market exchange.

- **`vequil/`** — core framework package
- **`trading/`** — internal orchestration layer
- **`web/`** — vequil.com landing page + dashboard

**Core philosophy:**
- Deterministic quant core (Kelly criterion + closed-form pricing) — *mathematically defensible, zero learned parameters*
- LLM analyst team runs async as **soft context only** — never overrides Kelly sizing, never in the execution hot path
- Paper → validate Sharpe ≥ 1.0 over 2+ weeks → then live

---

## Architecture

```
Feed Agent (Binance/Coinbase WS)  →  Feature Agent  →  Signal Bridge
Sports Agent (API-Football)        →  MatchEvent queue  →  (future: signal bridge)
                                                              ↓
                                                     Scanner Agent (Kalshi)
                                                              ↓
                                                        Risk Agent
                                                     (Kelly + pos limits + circuit breaker)
                                                              ↓
                                                    Execution Agent (paper/live → Kalshi)
```

**Key invariants — do not break these:**
1. `pricing.py` and `kelly.py` are pure math. No side effects. No learned parameters.
2. Every trade goes to SQLite. Full audit trail.
3. Paper mode is default. `EXECUTION_MODE=live` requires explicit opt-in.

---

## Exchange: Kalshi

Vequil trades exclusively on **Kalshi** — the CFTC-regulated US prediction market exchange.

**Kalshi SDK:** `kalshi_python_sync` installed. Base URL: `https://api.elections.kalshi.com/trade-api/v2`

**Auth:** RSA-PSS signature-based.
- `KALSHI_API_KEY` — key ID from Kalshi dashboard
- `KALSHI_PRIVATE_KEY_PATH` — path to RSA private key PEM file

**Market structure:**
- Markets are organized as: Series → Events → Markets
- Liquid markets fetched via `GET /series/{ticker}` → events → markets (not the raw `/markets` endpoint which returns low-volume combo markets)
- Key fields: `ticker`, `yes_ask_dollars`, `yes_bid_dollars`, `volume_fp`, `liquidity_dollars`, `close_time`

---

## Key Files

| File | Purpose |
|------|---------|
| `vequil/core/pricing.py` | Spot→probability (Black-Scholes N(d₂)) |
| `vequil/core/kelly.py` | Fractional Kelly with 0.25× cap, 4% min edge |
| `vequil/core/models.py` | Frozen dataclasses — Tick, Signal, KalshiMarket, Order, etc. |
| `vequil/pipeline.py` | Public `Pipeline` API — orchestrates evaluation |
| `vequil/paper.py` | Autonomous paper trading loop (Kalshi-wired) |
| `vequil/agents/scanner_agent.py` | Kalshi market matching + edge scoring |
| `vequil/agents/execution_agent.py` | Paper/live order placement via Kalshi |
| `trading/main.py` | Internal async orchestrator |

---

## Risk Controls (do not loosen without discussion)

| Control | Value |
|---------|-------|
| Kelly cap | 0.25× |
| Min edge | 4% |
| Min Kelly fraction | 1% of bankroll |
| Max concurrent positions | 5 |
| Max position size | $25 USDC |
| Max daily loss (circuit breaker) | $100 USDC |
| Max single exposure | 10% of bankroll |

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `API_FOOTBALL_KEY` | No | Live sports event data |
| `EXECUTION_MODE` | No | `paper` (default) or `live` |
| `BANKROLL_USDC` | No | Starting bankroll (default: 500.0) |
| `KALSHI_API_KEY` | Yes | Key ID from Kalshi dashboard |
| `KALSHI_PRIVATE_KEY_PATH` | Yes | Path to RSA private key PEM |

---

## Current Status

- [x] Full agent pipeline architecturally complete
- [x] Paper trading loop functional
- [x] `kalshi_python_sync` installed
- [x] RSA key pair generated (`~/.vequil/kalshi_private.pem` + `kalshi_public.pem`)
- [x] `KalshiMarket` model (replaces PolymarketMarket in all files)
- [x] `vequil/core/kalshi_client.py` — auth client with RSA-PSS signing
- [x] Kalshi scanner agent — autonomous scan loop via KalshiClient + Pipeline
- [x] Kalshi paper trading loop — fully ported
- [ ] **NEXT: Upload public key to Kalshi dashboard, set KALSHI_API_KEY in .env**
- [ ] Verify authenticated market fetch returns real liquid markets
- [ ] Run `vequil demo` / `python3 -m vequil.paper` end-to-end
- [ ] Kalshi live execution (after paper Sharpe ≥ 1.0)

---

## Session Handoff Notes

*Update this section when ending a session so the next agent has context.*

**Last worked on (2026-04-12, Antigravity -> Gemini):**
- **LLM/Anthropic Purge:** Completely removed `AnalystTeam`, `ANTHROPIC_API_KEY`, and `enable_analysts` from pipeline, paper trader, and cli. Research is now done explicitly in the IDE IDE agents (like Gemini/Antigravity). We are purely a deterministic Kelly-constrained engine now.
- **Kalshi Validation:** The `.env` was successfully populated with `KALSHI_API_KEY` (UUID) and `KALSHI_PRIVATE_KEY_PATH` (RSA PEM). Ran the `PaperTrader` directly against Kalshi; authentication works, signature is accepted, and paper pipeline is functionally sound (safely handled a rate-limit gracefully on rapid polling).
- **Cleared Old Context:** Purged the `AnalystTeam` architecture blocks from `CLAUDE.md`.

**NEXT STEP FOR CLAUDE TERMINAL:**
1. Wait for Phase 1 Data Capture Script to be approved and built.
2. We are migrating Phase 0 research (Lag/Maker feasibility) from Polymarket to Kalshi. Next effort is gathering continuous 1-5 second order book depth data on Kalshi live markets.
3. Keep an eye on Kalshi rate-limits as the new `kalshi_data_capture.py` is rolled out into `trading/research/`.

**trading/ directory:** Still mirrors old Polymarket code. Mirror vequil/ changes there when ready.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vequil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
