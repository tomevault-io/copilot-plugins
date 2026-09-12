## fomo-robinhood-radar

> Discovers Robinhood Chain memecoin traders (fomo.family + fresh-token holders), tracks their wallets on-chain, scores them with Claude, and serves signals / tokens / traders through a site, a Telegram bot and an HTTP API. The signals are meant to be acted on — an execution engine is planned — so nothing here should describe itself as read-only.

# FOMO Robinhood Radar

Discovers Robinhood Chain memecoin traders (fomo.family + fresh-token holders), tracks their wallets on-chain, scores them with Claude, and serves signals / tokens / traders through a site, a Telegram bot and an HTTP API. The signals are meant to be acted on — an execution engine is planned — so nothing here should describe itself as read-only.

**Naming.** The product is *FOMO Robinhood Radar* — the chain belongs in the name, on every page title and in the repo, because that is what people search for. The Python package stays `fomo_agent`: renaming it would touch every import for no user-visible gain. The CLI answers to both `fomo-radar` and `fomo-agent`.

**Start every session by reading `docs/STATUS.md`** — it has the current state, blockers, and the prioritized work list. Update it at the end of the session.

## Rules
- Never invent fomo.family endpoints. `sources/fomo.py` is filled only from `docs/fomo-endpoints.md` (phase 0 capture).
- Thresholds/intervals live in `config.py` + `.env`. No hardcoding.
- Every source must fail soft: one API down must not stop the loop.
- Codex budget (10k requests/month) still binds discovery and resolution; `cli init` prints a projection, check it before adding any Codex call. Tracking no longer spends it: `sources/rpc.py` covers the whole Robinhood roster in two `eth_getLogs` calls. Prefer a free source before reaching for Codex.
- Quote assets (USDG, WETH — `QUOTE_TOKENS` in `sources/rpc.py`) are never positions. Codex books swaps from the pool's side, so "sold X for USDG" arrives as a USDG buy; anything user-facing must exclude them.
- These Codex queries are plan-gated and unavailable: `filterWallets`, `detailedWalletStats`, `balances`. Working ones: `filterTokens`, `getTokenEvents`, `getTokenEventsForMaker`.
- Scoring must keep working without `ANTHROPIC_API_KEY` via the `score --export` / `--import` loop.
- Keep `pytest -q` green (offline fixture tests in `tests/`). Add a fixture + test for every new parser.
- No secrets in code or docs. `.env` is gitignored.

## Environment quirks (Windows)
- Run everything with `.venv/Scripts/python`; set `PYTHONIOENCODING=utf-8` for live runs (token names break cp1251).
- Bash heredocs with single quotes broke repeatedly here; edit files with Write/Edit or a python script.

## Commands
```
.venv/Scripts/python -m pytest -q
.venv/Scripts/python -m fomo_agent.cli init
.venv/Scripts/python -m fomo_agent.cli new-tokens --dry-run
.venv/Scripts/python -m fomo_agent.cli run --once
```

---
> Source: [cvxv666/fomo-robinhood-radar](https://github.com/cvxv666/fomo-robinhood-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
