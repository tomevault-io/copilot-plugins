## naas

> **Full component specs, data flow, DB schema, Redis usage, communication patterns, and design rationale**

# CLAUDE.md — NAAS Project Reference

## Important Subdocuments

### 🏗️ [docs/architecture/SYSTEM_ARCHITECTURE.md](docs/architecture/SYSTEM_ARCHITECTURE.md) - Detailed Architecture

**Full component specs, data flow, DB schema, Redis usage, communication patterns, and design rationale**

### 🤖 [docs/AI-AGENT-PRINCIPLES.md](docs/AI-AGENT-PRINCIPLES.md) - Behavioral Guidelines

**CRITICAL:** These principles apply to ALL Claude Code sessions and sub-agents

Contains operational principles for rigorous software engineering:
- Assumption surfacing before implementation (never silently guess requirements)
- Confusion management (stop and clarify inconsistencies)
- Simplicity enforcement (resist overcomplication, prefer boring solutions)
- Scope discipline (surgical precision, no unsolicited refactoring)
- Push back when warranted (not a yes-machine, challenge bad ideas respectfully)

**When to use:** EVERY session, EVERY task, EVERY agent. These behavioral guidelines work alongside project-specific technical constraints.

**Key principle:** "You are the hands; the human is the architect. Move fast, but never faster than the human can verify."

## What Is This?

**NAAS (Normalized Adaptive Access System)** — Enterprise IAM modernization platform providing unified, risk-based access control across OIDC, SAML, and LDAP identity systems.

**Tagline:** "Normalize once. Secure everywhere."

## Tech Stack

- **Backend:** Python 3.12+ / FastAPI 0.115+ / SQLAlchemy 2.0 (async) / Pydantic 2.10+
- **Frontend:** React 19 / TypeScript / Vite 6 / shadcn/ui / Tailwind / TanStack Query / React Flow / Recharts
- **Data:** PostgreSQL 17+ / Redis 7.4+ (Streams, Pub/Sub, caching)
- **Infrastructure:** Docker Compose / Keycloak 26+ (OIDC) / OpenLDAP 2.6+ / Prometheus 2.54+ / Grafana 11+
- **Logging:** Structlog (JSON structured with correlation IDs)
- **ML:** scikit-learn (Random Forest ensemble)

## Project Structure

```
naas/
├── docker-compose.yml
├── CLAUDE.md                       # THIS FILE
├── services/
│   ├── api-gateway/                # JWT auth, routing, WebSocket, rate limiting
│   ├── event-ingestion/            # Accept + validate login events, dual-write PG + Redis
│   ├── identity-normalization/     # OIDC/SAML/LDAP adapters → unified schema ★ KEY DIFFERENTIATOR
│   ├── signal-enrichment/          # IP reputation, geo, device, impossible travel
│   ├── risk-evaluator/             # Rule-based + ML scoring → allow/MFA/deny
│   ├── policy-management/          # YAML policy CRUD, versioning, shadow mode
│   ├── alert-service/              # High-risk event alerting (never on historical events)
│   └── persona-simulator/          # LLM-powered event generation (Claude/Ollama/mock fallback)
├── scripts/
│   └── train_bootstrap_model.py    # ML model bootstrap — generates random_forest.pkl
├── shared/
│   └── naas_shared/
│       ├── ml_features.py          # ML feature column ordering contract
│       └── simulation_tools.py     # Tool definitions for persona-simulator and MCP server
├── config/
│   └── normalization.yaml            # Normalization service config: per-attribute authority weights, attribute importance, cross-protocol enrichment source config
├── dashboard/                      # React SPA (5 tabs + floating simulator panel)
├── infrastructure/                 # Docker configs: postgres, redis, keycloak, openldap, monitoring
└── docs/
    ├── architecture/
    │   └── SYSTEM_ARCHITECTURE.md  # ★ FULL architectural reference — READ THIS FOR DETAILS
    ├── adr/                        # Architectural Decision Records
    └── guides/
```

## Event Pipeline (async, Redis Streams)

```
Ingestion → [login_events] → Normalization (+ LDAP enrichment for OIDC/SAML) → [normalized_events] → Enrichment → [enriched_events] → Risk Evaluator
                                                                                                                                         │
Alert Service ◄── [decisions Pub/Sub] ◄── Risk Evaluator                                                                                 │
Dashboard     ◄── [alerts Pub/Sub]    ◄── Alert Service                                                                                  │
Dashboard     ◄── [decisions Pub/Sub] ◄──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

## Key Commands

```bash
docker compose up              # Start all services
docker compose up -d           # Start detached
docker compose up -d --build   # Start detached, rebuilding local images (see note)
docker compose logs -f <svc>   # Tail service logs
docker compose ps              # Check service health
```

> **`--build` (locally-built images only):** `openldap` (`infrastructure/openldap/`) and every
> application service (e.g. `event-ingestion`) build from local Dockerfiles — a plain `up` reuses
> the cached image, so pass `--build` after changing their source. `postgres`/`redis`/`keycloak`
> pull pre-built images; `--build` is a no-op for them.
>
> **Schema changes need a volume wipe.** `infrastructure/postgres/init.sql` is the only schema
> source (no runtime DDL/migrations) and runs *only against an empty `postgres-data` volume*.
> Editing it does nothing to an existing volume. To pick up DDL changes (wipes data):
> ```bash
> docker compose down -v && docker compose up -d --build   # resets postgres + redis + ldap
> docker compose rm -sf postgres && docker volume rm naas_postgres-data && docker compose up -d postgres   # postgres only
> ```

## Python Virtual Environment

If `.venv/` exists, activate it (`source .venv/bin/activate`) before running any Python commands. Use `python` (not `python3`) after activation.

## Key Conventions

- **Every service:** Own `Dockerfile`, `requirements.txt`, `app/main.py` (FastAPI)
- **Validation:** Pydantic models for all API schemas
- **Logging:** Structlog (JSON) with a per-request `correlation_id` bound by ASGI middleware; `event_id` correlates events across the stream pipeline
- **Stream consumers:** `XREADGROUP` with consumer groups; ACK only after success
- **Fail-safe:** Unknown risk → DENY; service down → CHALLENGE
- **Metadata on every event:** `source`, `is_synthetic`, `is_historical`, `protocol`
- **Cross-protocol enrichment:** Identity Normalization queries OpenLDAP for OIDC/SAML events to merge directory attributes with token claims. Configurable unified schema correlation field (default: `primary_email`; adapter reverse-maps to LDAP attribute internally). Cached in Redis (60s TTL). Graceful degradation on failure. LDAP events skip enrichment. Config in `config/normalization.yaml` under `enrichment.sources.ldap`.
- **Markdown files:** Preserve all Unicode characters (emojis, box-drawing, arrows) as-is — never replace with ASCII equivalents
- **LLM Integration:** Persona Simulator uses configurable LLM provider (Claude API → Ollama → mock). Set via `LLM_PROVIDER` env var. Default: `mock` (no API keys needed). Events submitted via EventSink abstraction.
- **Shared tools:** `shared/naas_shared/simulation_tools.py` contains tool definitions and executor used by persona-simulator (internal) and MCP server (external, P2).
- **Policy Model:** Hybrid scoring — `signal_weights` (4 continuous signals: ip_reputation_risk, normalization_risk, failed_login_risk, login_recency_risk) + `conditions` (boolean expressions evaluated by Python ast-based safe evaluator). Expression language supports AND/OR/NOT/IN operators across 5 namespaces (user, device, signals, time, event).
- **ML Model:** Bootstrap script at `scripts/train_bootstrap_model.py` generates `random_forest.pkl` from synthetic distribution profiles. Feature vector (16 columns) defined in `shared/naas_shared/ml_features.py` (currently a placeholder — populated by Spec 3) — shared between training and inference. Model labels are independent of rule-based scoring.

## Agentic Pipeline

This project is implemented via an automated agentic pipeline managed by a `pipeline-orchestrator` skill. If you are a worker agent invoked by the orchestrator, these rules apply:

- **You are a stateless specialist.** Do your assigned work, produce your artifacts, return your summary. You do not manage pipeline lifecycle.
- **Do not read or write pipeline state files** (`state.json`, `chunks.json`, pipeline logs). The orchestrator owns these exclusively.
- **Do not run git commands.** No `git add`, `git commit`, `git push`, `git checkout`. SCM is the orchestrator's responsibility.
- **Pipeline mode vs. manual mode:** When invoked via Task (pipeline mode), do not use `AskUserQuestion` — state problems clearly in your response and let the orchestrator handle escalation. When invoked directly by the developer (manual mode), use `AskUserQuestion` freely.
- **Your context comes from the Task prompt.** Don't go looking for pipeline artifacts or other agents' output to figure out what you should be doing.

Pipeline details, phase definitions, and inter-agent contracts live in `.claude/pipeline/` and `docs/Agentic_Workflow_Implementation_Guide.md`. The `pipeline-orchestrator` skill (invoked as `/pipeline-orchestrator`) is the sole entry point for automated pipeline runs.

## Git and GitHub Conventions

- **Branches:** `feature/<short-description>`, `fix/<short-description>`, `chore/<short-description>`
- **Commits:** Use conventional commits — `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- **PR titles:** Same prefix as commits, imperative mood, under 70 chars

---
> Source: [twoffer/naas](https://github.com/twoffer/naas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
