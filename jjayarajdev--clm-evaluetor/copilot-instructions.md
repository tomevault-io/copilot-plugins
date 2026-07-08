## clm-evaluetor

> This file provides guidance to Codex when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

## Project Overview

**Evaluetor** is an AI-native contract lifecycle management (CLM) platform that combines contract intelligence (AI-powered extraction and analysis) with relationship governance (KPI perception scoring, business relationship management). Built to disrupt legacy CLM vendors through rapid time-to-value, agentic AI workflows, and modern architecture.

**Current State:** Phases 0-10 complete. ~315 API endpoints, 41 routers, 36 services, 9 AI agents, 48 model files, 55+ database tables.

## Technical Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.11+ / FastAPI (async) |
| Frontend | React / TypeScript / Vite |
| Database | PostgreSQL + ChromaDB (vectors) |
| AI | OpenAI GPT-4o, Agent Squad orchestration |
| Package Mgr | UV (backend), npm (frontend) |
| Observability | Langfuse via OpenTelemetry |
| Deployment | Docker Compose (local + AWS EC2) |

## Common Commands

```bash
# Backend
cd backend && uv sync                                    # Install deps
cd backend && uv run pytest                              # Run tests
cd backend && uv run uvicorn app.main:app --reload       # Dev server (port 8000)

# Frontend
cd frontend && npm install                               # Install deps
cd frontend && npm test                                   # Run tests
cd frontend && npm run dev                                # Dev server (port 3000/3002)

# Seeding
cd backend && uv run python -m scripts.seed_data                        # Core data
cd backend && uv run python -m scripts.seed_relationship_governance     # Governance data

# Docker (Production/AWS)
cd deploy && docker-compose -f docker-compose.prod.yml up -d --build backend   # Deploy backend
cd deploy && docker-compose -f docker-compose.prod.yml build --no-cache frontend && docker-compose -f docker-compose.prod.yml up -d frontend  # Deploy frontend
```

## Demo Credentials

| Tenant | Username | Password | Role |
|--------|----------|----------|------|
| Acme Corp | admin | admin123 | Admin |
| Acme Corp | legal | legal123 | Legal |
| TechStart | techstart_admin | admin123 | Admin |
| LegalCo | legalco_admin | admin123 | Admin |
| (System) | superadmin | admin123 | Super Admin |

## Critical Rules

### Database Enums
PostgreSQL enums are immutable. When adding values to Python enums, ALWAYS create an Alembic migration:
```python
def upgrade() -> None:
    op.execute("ALTER TYPE your_enum ADD VALUE IF NOT EXISTS 'new_value'")
```
Use `PG_ENUM(*[e.value for e in EnumClass], name='enumname', create_type=False)` for enum columns — never `Enum(PythonStrEnum)` which sends uppercase names instead of lowercase values.

### Multi-Tenant Isolation
- Every query MUST filter by `tenant_id` using `apply_tenant_filter()` or `CurrentTenantId` dependency
- Super admin has `tenant_id=null` — handle explicitly when creating resources across tenants
- All routers use `/api/` prefix (e.g., `/api/contracts`, `/api/organizations`)

### SQLAlchemy Async Patterns
- Never use `begin_nested()` savepoints — causes `MissingGreenlet` errors
- Never use separate sessions for related operations — causes deadlocks
- `lazy="dynamic"` relationships cannot use `selectinload` — use separate queries

### User Management
- Usernames cannot contain spaces (used for login)
- Super admin must specify `tenant_id` in request body when creating users

## Architecture

### Layered Pattern
```
Routers (API) → Services (Business Logic) → Models (SQLAlchemy) → PostgreSQL
                                           → Agents (AI) → OpenAI / ChromaDB
```

### AI Pipeline (Contract Upload)
Upload → Parse (PDF/DOCX) → Chunk → Embed (ChromaDB) → Extract Metadata → Detect Risks → Extract Clauses → Extract Obligations → Detect Renewals → Extract SLAs → Auto-Link Detection

### 9 AI Agents
1. Metadata Extraction — parties, dates, values
2. Contract Q&A — RAG-powered answers
3. Risk Detection — 10 risk categories
4. Obligation Tracking — deadlines, consequences
5. Clause Extraction — 17 clause types
6. Renewal Monitoring — auto-renewal, notice periods
7. SLA Extraction — metrics, targets, penalties
8. Schema Extraction — 15 contract types, 1,235 fields
9. Intent Router — query routing with LLM visualization

### Relationship Governance (Evaluetor)
Organizations → Business Relationships → KPIs → Perception Scores → Gap Analysis → Improvement Points → Surveys

## Key Directories

| Path | Purpose |
|------|---------|
| `backend/app/routers/` | 41 API routers |
| `backend/app/models/` | 48 SQLAlchemy models |
| `backend/app/services/` | 36 service modules |
| `backend/app/agents/` | AI agent definitions |
| `backend/scripts/` | Seed, reindex, test scripts |
| `frontend/src/pages/` | 30+ page components |
| `frontend/src/components/` | Shared UI components |
| `frontend/src/lib/api.ts` | API client (all backend calls) |
| `deploy/` | Docker Compose configs |
| `docs/` | Architecture, API, data model docs |

## Reference Documents

- `docs/ARCHITECTURE_OVERVIEW.md` — System architecture
- `docs/API_DOCUMENTATION.md` — ~315 endpoints
- `docs/DATA_MODEL.md` — ER diagrams
- `docs/VALIDATION_TEST_PLAN.md` — Feature validation checklist
- `docs/PRODUCT_VISION_AND_ROADMAP.md` — Product roadmap

## Deployment

- **AWS SSH:** `ssh -i ~/.ssh/clm-demo-key.pem ec2-user@52.21.204.211`
- **IMPORTANT:** Always use `-f docker-compose.prod.yml` flag on AWS
- **IMPORTANT:** `docker-compose up -d` alone does NOT rebuild — use `--build`
- **Frontend** requires `--no-cache` to pick up source changes

---
> Source: [jjayarajdev/clm-evaluetor](https://github.com/jjayarajdev/clm-evaluetor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
