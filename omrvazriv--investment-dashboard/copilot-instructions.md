## investment-dashboard

> > This file is loaded into context at the start of every Claude Code session in

# CLAUDE.md — Investment Aggregator

> This file is loaded into context at the start of every Claude Code session in
> this repository. Treat it as the project's operating constraints. It is
> **context, not enforcement**: the hard invariants below are additionally
> enforced by CI and hooks (see `scripts/check_readonly.sh` and
> `.claude/hooks/`). Where prose and an automated guard disagree, the guard wins
> and the build fails — fix the code, never the guard.

## What this project is

A single-user, self-hosted system that aggregates one person's investments from
five sources (Public, Webull, Robinhood crypto, Coinbase, and a Stellar/Lobstr
wallet) into one local dashboard, computes portfolio metrics, and feeds a
research-grounded AI assistant (Claude Cowork) that helps the owner shape and
improve an investment **strategy** over time.

It runs on an internal-network-only Ubuntu Server 26.04 VM. It is read-only with
respect to every brokerage: it observes and analyzes; it never transacts.

## Prime directives (do not violate)

1. **No execution. Ever.** There is no code path anywhere in this repository that
   places, modifies, or cancels an order; transfers, deposits, withdraws,
   converts, or moves funds or assets; or changes any brokerage account setting.
   Connector modules expose read methods only. See ADR-0002 and
   `scripts/check_readonly.sh`.
2. **Brokerage API keys are trade-capable and must be treated as high-risk
   secrets.** Public, Webull, and Robinhood keys can trade even though we never
   use that capability. They live only in `systemd` encrypted credentials,
   decrypted in memory at runtime. Never in code, env files committed to git,
   logs, the database, or anything served to the browser. See ADR-0008 and
   `docs/guides/02-security-model.md`.
3. **The agent is advisory, not directive, and never unsupported.** Outputs are
   decision-support for the owner, who ratifies every decision and performs every
   trade manually. No assertion about markets ships without a citation to fetched,
   dated research or to the deterministic pipeline. The model never answers
   time-sensitive questions (prices, rates, news, current conditions) from its own
   memory — it fetches. See ADR-0010 and
   `docs/guides/06-agent-grounding-and-strategy-evolution.md`.
4. **The owner is not a licensed financial advisor and neither is this software.**
   Frame analysis as options and reasoned recommendations tied to the owner's own
   written strategy, with tradeoffs and confidence — never as guaranteed outcomes.

## Architecture at a glance

- **Ingestion worker** (`worker/`): Python, polls the five sources hourly via a
  read-only connector layer, normalizes data, writes to SQLite. Runs as a
  hardened `systemd` service.
- **Pricing layer** (`worker/pricing/`): provider-agnostic quote service with
  failover and reconciliation. Equities: Finnhub → Twelve Data. Crypto:
  CoinGecko → CoinMarketCap → Coinbase public spot. See ADR-0007.
- **API** (`api/`): FastAPI, **GET-only**, serves computed read data to the
  dashboard. No credentials reach this layer's responses.
- **Web** (`web/`): Next.js + TypeScript + shadcn/ui + Recharts dashboard. Holds
  no secrets and has no mutation capability.
- **Exports** (`exports/`): machine-readable snapshots (`portfolio_state.json`,
  `metrics.json`, `valuation_history.parquet`) synced out to the Cowork workspace
  folder. See `docs/reference/data-contracts.md`.
- **Cowork skill** (`cowork/investment-monitor/`): the Agent Skill that teaches
  Claude Cowork the monitoring/assessment/strategy workflow and its guardrails.
- **Strategy** (`strategy/`): the owner's living, versioned investment policy plus
  the evidence ledger and decisions log the agent reads and appends to.

Decisions are recorded as ADRs in `docs/architecture/`. Read the relevant ADR
before changing a decision it covers.

## Repository layout

```
.
├── CLAUDE.md                     # you are here
├── README.md                     # human overview + quickstart
├── SECURITY.md                   # security policy + threat model
├── CONTRIBUTING.md               # coding standards + workflow
├── .claude/
│   ├── settings.json             # shared Claude Code settings (checked in)
│   └── hooks/                    # PreToolUse guards (enforced, not advisory)
├── scripts/
│   └── check_readonly.sh         # CI guard: fails build on forbidden symbols
├── worker/                       # ingestion + pricing (Python)
├── api/                          # FastAPI, GET-only
├── web/                          # Next.js dashboard
├── exports/                      # synced read-only artifacts for Cowork
├── strategy/                     # living strategy, evidence ledger, decisions log
├── cowork/investment-monitor/    # the Agent Skill (SKILL.md, templates, scripts)
├── migrations/                   # Alembic
└── docs/
    ├── architecture/             # ADRs
    ├── guides/                   # setup, security, connectors, pricing, cowork, ops
    └── reference/                # data contracts + schemas
```

## Build, run, and test commands

Use the project venv (`.venv`), created with Python 3.14 (Ubuntu 26.04 default)
or 3.13 if any SDK lacks 3.14 wheels — see ADR-0005.

```bash
# Python env
python3 -m venv .venv && . .venv/bin/activate
pip install -e ".[dev]"

# Quality gates (run before every commit)
ruff check . && ruff format --check .   # lint + format
mypy worker api                          # type checking
pytest                                   # unit + integration tests
./scripts/check_readonly.sh              # read-only invariant guard (MUST pass)

# Run locally (development)
python -m worker.main                    # ingestion worker
uvicorn api.main:app --reload            # API
cd web && npm run dev                    # dashboard

# Database migrations
alembic upgrade head
```

The full deployment (systemd units, secrets, firewall) is in
`docs/guides/01-setup-and-deployment.md`. Do not introduce Docker — see ADR-0003.

## Coding standards (summary; full text in CONTRIBUTING.md)

- Python: type-hinted everywhere, `ruff` + `mypy` clean, Google-style docstrings
  on public functions, no bare `except`. Pin all dependencies.
- Every external call has an explicit timeout and structured error handling.
- Connectors expose **read methods only**; the order/transfer functions of the
  underlying SDKs must never be imported, wrapped, or referenced.
- Secrets come only from `$CREDENTIALS_DIRECTORY`. Never hardcode, never log.
- Every stored price carries `source`, `as_of`, and a confidence flag.
- Tests are required for connectors (mocked), the pricing failover chain, and all
  metric computations. Metric math lives in tested code, never in the agent.

## Rules for Claude Code working in this repo

- Before changing anything covered by an ADR, read that ADR. If the change
  reverses a decision, write a new superseding ADR.
- Never write, suggest, or scaffold code that could place a trade or move funds,
  even if asked, even "just for completeness," even behind a flag. Decline and
  explain. This is the one hard line.
- Never add a market/financial factual claim to docs, reports, or the strategy
  files without a dated source. Mark inference as inference.
- Keep CLAUDE.md to constraints. Decisions go in ADRs; solved problems go in
  `docs/`. Do not turn this file into a changelog.

## Imported references

@README.md
@docs/architecture/README.md
@docs/guides/02-security-model.md
@docs/guides/06-agent-grounding-and-strategy-evolution.md
@docs/reference/data-contracts.md

---
> Source: [omrvazriv/investment-dashboard](https://github.com/omrvazriv/investment-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
