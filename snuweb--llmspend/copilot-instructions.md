## llmspend

> LLMSpend is a developer tool that tracks LLM API costs per feature, per user, per model.

# CLAUDE.md — LLMSpend Project Specification
# Read this file at the start of every session before doing any work.

## What This Project Is
LLMSpend is a developer tool that tracks LLM API costs per feature, per user, per model.
Developers install a Python SDK (`pip install llmspend`) that wraps their AI client with 2 lines of code.
The SDK logs every API call's cost, tokens, latency, and metadata to a dashboard with alerts.

**One sentence:** "Stripe Dashboard for your AI spend."

**What we are NOT:** We are not an observability platform, not a prompt manager, not an eval framework,
not a gateway/proxy. We do ONE thing — show developers where their AI money goes and alert them
when something is wrong.

## The Problem We Solve
Provider dashboards (Anthropic, OpenAI) show total spend but NOT:
- Which feature/endpoint in your app costs the most
- Which user is burning through your budget
- Real-time alerts before the bill arrives
- Per-request cost tied to application logic
- Cost trends and anomaly detection

Developers find out they overspent when the invoice arrives. We show them in real-time.

## Stack
- **SDK:** Python 3.10+, zero dependencies beyond `httpx` for async log shipping
- **Backend:** Python 3.12, FastAPI, SQLAlchemy (async), asyncpg, uvicorn
- **Frontend:** React (JSX, not TypeScript), Vite
- **Database:** PostgreSQL 16
- **Hosting:** DigitalOcean
- **Alerts:** Email, Telegram bot, Discord webhook, Slack webhook
- **NO** TypeScript, NO Next.js, NO Tailwind (plain CSS), NO Go, NO Django

## Project Structure
```
/home/sarif/projects/llmspend/
  sdk/                          — Python SDK (pip package)
    llmspend/
      __init__.py               — Public API: wrap(), configure()
      wrapper.py                — Client wrapper (Anthropic, OpenAI)
      transport.py              — Async log shipping to backend
      pricing.py                — Token-to-cost calculation (per model)
      local.py                  — Local SQLite fallback (self-hosted mode)
    setup.py / pyproject.toml
    README.md
  backend/                      — FastAPI server (hosted version)
    llmspend_api/
      app.py                    — CORS, middleware, lifespan
      routes/
        ingest.py               — POST /v1/events (receives SDK logs)
        dashboard.py            — GET endpoints for frontend
        alerts.py               — Alert configuration CRUD
        auth.py                 — API key auth, project management
      db/
        connection.py           — async_session factory
        models.py               — SQLAlchemy ORM models
      alerting/
        engine.py               — Alert evaluation (budget, spike, anomaly)
        channels.py             — Email, Telegram, Discord, Slack dispatch
      config.py                 — Settings (env vars)
    main.py                     — Entry point
  frontend/                     — React SPA (Vite)
    src/
      App.jsx                   — Main app, routing
      pages/
        Dashboard.jsx           — Cost overview, charts
        ModelBreakdown.jsx      — Per-model cost analysis
        FeatureCosts.jsx        — Per-feature/endpoint costs
        Alerts.jsx              — Alert configuration
        Settings.jsx            — Project settings, API keys
      components/
        CostChart.jsx           — Time-series cost chart
        UsageTable.jsx          — Tabular usage data
      api.js                    — API client
  docs/                         — Documentation site (later)
```

## SDK Design — CRITICAL RULES

### How it works
```python
# Developer's code — 2 lines to integrate
from llmspend import monitor
client = monitor.wrap(anthropic.Anthropic(), project="my-app")

# Everything else works exactly the same
response = client.messages.create(
    model="claude-haiku-4-5-20251001",
    max_tokens=500,
    messages=[{"role": "user", "content": "Hello"}],
    # Optional: tag this call for per-feature tracking
    metadata={"feature": "chatbot", "user_id": "u_123"}
)
```

### SDK Rules
1. **ZERO impact on the developer's code** — wrap() returns an object with the identical API
2. **NEVER modify, delay, or interfere with API calls** — calls go directly to the provider
3. **NEVER store prompts or responses by default** — only metadata (tokens, cost, model, latency)
4. **Logging is async and non-blocking** — if our server is down, their app is unaffected
5. **NEVER crash the developer's app** — all SDK code wrapped in try/except, fails silently
6. **NEVER require the developer to change their existing code** beyond the 2 setup lines
7. **Works offline** — falls back to local SQLite if no backend configured

### What the SDK logs per call
```json
{
  "timestamp": "2026-03-10T12:00:00Z",
  "provider": "anthropic",
  "model": "claude-haiku-4-5-20251001",
  "tokens_in": 150,
  "tokens_out": 89,
  "cost_usd": 0.000143,
  "latency_ms": 432,
  "status": "success",
  "feature": "chatbot",
  "user_id": "u_123",
  "project": "my-app"
}
```

### What the SDK NEVER logs (unless explicitly opted in)
- Prompt content
- Response content
- API keys
- System prompts
- Tool definitions

## Pricing Model (per model, USD)
Keep a static map updated manually. Source of truth: provider pricing pages.
```
anthropic:
  claude-opus-4-6:          input=$15.00/MTok  output=$75.00/MTok
  claude-sonnet-4-6:        input=$3.00/MTok   output=$15.00/MTok
  claude-haiku-4-5-20251001: input=$0.80/MTok  output=$4.00/MTok

openai:
  gpt-4o:                   input=$2.50/MTok   output=$10.00/MTok
  gpt-4o-mini:              input=$0.15/MTok   output=$0.60/MTok
  gpt-4.1:                  input=$2.00/MTok   output=$8.00/MTok
  gpt-4.1-mini:             input=$0.40/MTok   output=$1.60/MTok
  gpt-4.1-nano:             input=$0.10/MTok   output=$0.40/MTok
  o3:                       input=$2.00/MTok   output=$8.00/MTok (non-reasoning)
  o4-mini:                  input=$1.10/MTok   output=$4.40/MTok
```

## Business Model — Open Core (Phase 1: open source only)

### Free (open source, MIT license)
- Python SDK on PyPI
- Local SQLite logging + CLI (`llmspend stats --last 7d`)
- Self-hosted dashboard (React + FastAPI)
- Unlimited calls, forever free

### Paid (hosted cloud, LATER — after 4 months)
- Hosted dashboard at llmspend.dev
- Team sharing
- Alerts (Telegram, Discord, Slack, email)
- 90-day history, cost forecasting, anomaly detection
- Budget caps and notifications
- $19/mo solo, $49/mo team

## Data Integrity Rules
1. **NO fake numbers** — all costs calculated from real token counts and real pricing
2. **NO estimated costs shown as exact** — if pricing is unknown for a model, show "unknown model"
3. **NO data collection beyond what's documented** — privacy is a selling point
4. **NO vendor lock-in** — self-hosted mode must always work fully offline

## Current Phase
Phase 1: Build the SDK + local dashboard. Open source on GitHub.
No paid features, no hosted version, no Stripe, no user accounts yet.

## Domain
- Website: llmspend.dev (not registered yet)
- PyPI: llmspend (available)
- GitHub: TBD

## Deployment (later)
- DigitalOcean droplet for hosted backend
- Same patterns as SwarmIntellect (FastAPI + PostgreSQL + nginx)
- Separate from SwarmIntellect infrastructure entirely

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/snuweb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/snuweb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
