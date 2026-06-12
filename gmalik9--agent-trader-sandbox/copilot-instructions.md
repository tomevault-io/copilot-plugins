## agent-trader-sandbox

> This repo runs LLM-driven trading agents. **It can never place a live,

# AGENTS.md — Safety guarantees, env vars, kill-switch

This repo runs LLM-driven trading agents. **It can never place a live,
real-money order.** This file documents the guarantees that make that true.

## Hard guarantees

1. **No live-broker code path.** The two `BrokerBase` implementations are:
   - `SandboxBroker` — purely local; cannot reach the internet for execution.
     "Fills" come from cached yfinance bars.
   - `AlpacaPaperBroker` — delegates to the upstream `stock-recommender` MCP
     server, which is itself hard-coded to `https://paper-api.alpaca.markets`
     and asserts `ALPACA_PAPER=true` and that the account number starts with
     `PA`.
2. **DualBroker is paper-only by construction** — it composes the two above
   and has no other backend.
3. **Kill-switch.** A row in `settings(key='kill_switch', value='on')` halts
   every agent within one tick (≤ 5 s). The Streamlit Overview tab flips it.
4. **Caps.** Per-order USD cap and per-symbol % of equity cap enforced inside
   `SandboxBroker` and via the upstream MCP for the Alpaca leg.
5. **Blocklist.** Leveraged / inverse / volatility ETFs blocked at the
   broker layer.
6. **Audit.** Every order, fill, cash move, and agent run is written to
   `data/sandbox.sqlite`. The Alpaca leg is additionally audited by the
   upstream repo's own SQLite log.

## Environment variables

| Var | When required | Purpose |
|---|---|---|
| `LLM_PROVIDER` | always (default `github`) | `github` \| `openai` \| `anthropic` |
| `LLM_MODEL` | optional | provider-specific model id |
| `GITHUB_TOKEN` | if `LLM_PROVIDER=github` | PAT with GitHub Models access |
| `OPENAI_API_KEY` | if `LLM_PROVIDER=openai` | |
| `ANTHROPIC_API_KEY` | if `LLM_PROVIDER=anthropic` | |
| `BROKER_BACKEND` | always (default `dual`) | `sandbox` \| `alpaca_paper` \| `dual` |
| `ALPACA_API_KEY_ID` | if backend ∈ {`alpaca_paper`,`dual`} | |
| `ALPACA_SECRET_KEY` | same | |
| `ALPACA_PAPER` | same; must equal `true` | |
| `STOCK_REC_MCP_TRADING_ENABLED` | same; must equal `true` | unlocks upstream WRITE tools |
| `STOCK_REC_MAX_ORDER_USD` | optional | per-order cap on Alpaca leg |
| `STOCK_REC_MAX_SYMBOL_PCT` | optional | per-symbol % cap on Alpaca leg |
| `SHORT_TERM_TRADER_PATH` | always | absolute path to sibling repo |
| `STOCK_RECOMMENDER_PATH` | always | absolute path to sibling repo |
| `CAPITAL_TOTAL` | optional (default `100000`) | |
| `SPLIT_DAY_PCT` | optional (default `30`) | day-trader's share |
| `FINNHUB_API_KEY` | optional | passed through to MCP subprocesses |
| `ALPHAVANTAGE_API_KEY` | optional | passed through to MCP subprocesses |

## Kill-switch

Flip from the Overview tab, or directly:

```sql
UPDATE settings SET value='on', updated_at=datetime('now')
 WHERE key='kill_switch';
```

The next scheduler tick will short-circuit with
`agent_runs.status='halted'`.

---
> Source: [gmalik9/agent-trader-sandbox](https://github.com/gmalik9/agent-trader-sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
