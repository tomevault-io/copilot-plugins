## minirouter

> This file helps AI coding agents understand the MiniRouter codebase. It

# MiniRouter — Agent Guide

This file helps AI coding agents understand the MiniRouter codebase. It
replaces the legacy `.claude/` and `.agents/` skill directories (removed).
Agents should start here, then read the relevant `docs/` file for details.

## Project structure

```
src/
├── auth/                # API Key authentication & user identity
├── config/              # .env loading & runtime configuration
├── context/             # Optional Headroom context optimization
├── db/                  # SQLite, migrations, query layer
├── protocols/           # OpenAI / Anthropic request normalization
├── providers/           # Slot env vars, channel selection, upstream adapters
├── router/              # 14-dimension rule routing engine
├── routing/             # Feature extraction & routing debug receipts
└── server/              # Hono HTTP API & SSE usage collection

admin/
└── dashboard.html       # Admin control panel (users, keys, channels, usage)

models/
├── seed-data.json       # Pricing & benchmark seed data (53 models)
├── seed-models.ts       # Import script: npm run seed:models
├── dashboard.html       # Model comparison table (public, /models/dashboard)
└── README.md            # Brief model data guide

scripts/                 # Development & ops tools (not in Docker image)
├── today.mjs            # Daily dashboard (direct SQLite query)
├── local-smoke.mjs      # Local smoke test
├── cache-usage-probe.mjs / stream-cache-probe.mjs
└── README.md

tools/                   # Model score import utilities (not in Docker image)
├── import-llm-stats.mjs / import-superclue-scores.mjs
├── scope-model-scores.mjs / score-utils.mjs + .test.mjs
├── model-benchmark-aliases.mjs / superclue-model-aliases.mjs
└── README.md

archive/                 # Dead code (registry, systemd scripts, one-time tools)
├── registry.ts          # Old hardcoded model registry (replaced by SQLite)
├── extract-models.mjs   # One-time extraction tool
├── update-log.md        # One-time model update log (2026-07-03)
├── analyze-routing.mjs / audit-*.mjs / usage-by-window.mjs
├── headroom-cache-experiment.mjs
└── README.md

deploy/
├── deploy.sh            # Bash Docker deploy script (SSH + docker compose)
├── deploy.mjs           # Node.js SSH deploy script (ssh2)
└── nginx-minirouter.conf
```

## Where to look

### How routing works → `docs/routing-strategy.md`

The 14-dimension classifier, tier boundaries, slot selection, and all
configuration parameters. README has a quick summary; this doc has the
full strategy.

### How to deploy → `README.md` (Docker section)

Docker Compose is the recommended deployment method. See also:
- `docker-compose.yml` — runtime config
- `deploy/deploy.sh` — upload + deploy script
- `deploy/nginx-minirouter.conf` — reverse proxy template

### Environment variables → `.env.example`

All configurable variables with sensible defaults in comments. The code
reads them via `src/config.ts` → `getConfig()`.

### Routing tuning → `README.md` (Routing tuning section)

Parameters that control SIMPLE / MEDIUM / COMPLEX / REASONING tier ratios.
Edit in `docker-compose.yml` → `environment:` and `docker compose up -d`.

### Database queries → `docs/db-queries.md`

SQLite schema, query recipes for usage logs, spend tracking, user management.

### Admin dashboard → `admin/dashboard.html`

Web UI for managing users, API keys, provider channels, and usage logs.
Served at `/admin/dashboard`. First-time setup at `POST /setup`.

### Model pricing & benchmarks → `docs/model-pricing.md`

Price comparison table, capability rankings, and recommended model pools.

### Model registry maintenance → `docs/model-maintenance/update-model-registry.md`

How to update pricing, benchmarks, and model scores.

### Headroom context optimization → `docs/headroom.md`

External proxy + local tail compression for long contexts.

### Infrastructure management → `docs/infra-management-design.md`

Channel management, provider instances, cost tracking architecture.

### Roadmap → `docs/roadmap.md`

What's planned: Headroom integration, admin dashboard, multi-tenancy, etc.

## Key files to read first

| File | Why |
|------|------|
| `src/router/config.ts` | DEFAULT_ROUTING_CONFIG + env var overrides |
| `src/providers/env.ts` | Slot selection logic |
| `src/router/rules.ts` | 14-dimension scoring |
| `src/server/routes/chat.ts` | OpenAI-compatible request handling |
| `src/server/routes/anthropic-messages.ts` | Anthropic Messages API handling |
| `src/server/routes/channel-execution.ts` | Channel failover logic |
| `admin/dashboard.html` | Admin control plane web UI |
| `models/seed-data.json` | Model pricing & benchmark source of truth |

---
> Source: [lpffernando/MiniRouter](https://github.com/lpffernando/MiniRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
