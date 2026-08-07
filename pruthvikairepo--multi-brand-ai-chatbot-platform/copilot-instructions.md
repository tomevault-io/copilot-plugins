## multi-brand-ai-chatbot-platform

> - **Project**: Multi-Brand AI Chatbot Platform for Skincare Brands

# Multi-Brand AI Chatbot Platform - Claude Code Project Guide

## Project Identity
- **Project**: Multi-Brand AI Chatbot Platform for Skincare Brands
- **Client**: Skincare brand owner managing multiple brands
- **Developer**: Keshav Infotech
- **SRS**: 31-page consolidated SRS (Sections 1-28), fully approved

## Architecture Overview
- **Backend**: Python / FastAPI (async)
- **Database**: PostgreSQL + pgvector extension (vector search)
- **Task Queue**: Celery + Redis
- **AI Engine**: Claude API (Sonnet) via Anthropic SDK with RAG
- **Embeddings**: Voyage AI or OpenAI Embeddings API
- **File Storage**: AWS S3
- **Admin Panel**: React.js + Tailwind CSS
- **Chat Widget**: React embeddable component + WebSocket
- **Auth**: JWT + bcrypt, RBAC (Super Admin / Admin)
- **Hosting**: AWS (ECS) or GCP (Cloud Run)

## Core Concept: Brand Isolation
This is a **multi-tenant** system where each brand is a fully isolated tenant.
- Every database query MUST filter by `brand_id`
- Vector DB uses brand-isolated namespaces (`brand_{id}_products`, etc.)
- S3 uses brand-scoped bucket paths
- Admin users are scoped to assigned brands (unless Super Admin)
- **Zero cross-contamination** is the #1 architectural requirement

## Key Documents
- `SCOPE_OF_WORK.md` — Complete backend/frontend/database scope
- `CLIENT_REQUIREMENTS.md` — What client must provide
- `BACKEND_DEVELOPMENT_GUIDE.md` — Step-by-step backend development guide (Phases A-L)
- `SRS_Multi_Brand_AI_Chatbot_Complete_Final.pdf` — Full SRS (source of truth, 31 pages)
- `SRS_Complete_Final (1).pdf` — Original SRS (16 pages, subset of above)

## Development Phases
Refer to `BACKEND_DEVELOPMENT_GUIDE.md` for detailed instructions per phase.
```
Phase A: Foundation & Infrastructure     → Project setup, Docker, config
Phase B: Core Data Models & CRUD         → All DB models, migrations, admin CRUD APIs
Phase C: Authentication & RBAC           → JWT, login, roles, permissions, audit trail
Phase D: AI Engine & RAG Pipeline        → Embeddings, vector search, Claude API, compliance filter
Phase E: Tone, Compliance & Rules Engine → Recommendation engine, skin quiz, session personalization
Phase F: Chat Widget Backend & WebSocket → WebSocket endpoint, session mgmt, widget config API
Phase G: Input Moderation & Security     → Moderation pipeline, prompt injection defense, rate limiting
Phase H: Lead Capture & Bot Protection   → Lead forms, reCAPTCHA, honeypot, encrypted storage
Phase I: Logging, Analytics & Observability → All log types, analytics aggregation, data masking
Phase J: Admin Panel API Completion      → Prompt versioning, overrides, image styles, caching
Phase K: Secret Management & Cost Control → AES-256 secrets, per-brand key resolution, cost tracking
Phase L: Testing, QA & Deployment        → Integration tests, Docker deploy, CI/CD
```

## Project Structure
```
backend/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── config.py            # Pydantic settings
│   ├── dependencies.py      # DI: DB session, current user, brand context
│   ├── api/v1/              # All API route modules
│   ├── models/              # SQLAlchemy ORM models
│   ├── schemas/             # Pydantic request/response schemas
│   ├── services/            # Business logic (ai, rag, embedding, tone, compliance, etc.)
│   ├── core/                # Security, permissions, rate limiter, cache, exceptions
│   ├── db/                  # Database session factory, base model
│   ├── tasks/               # Celery async tasks (embedding, cleanup)
│   └── middleware/          # Brand context, rate limiting, logging
├── alembic/                 # Database migrations
├── tests/                   # Test suite
├── Dockerfile
├── docker-compose.yml
└── requirements.txt

frontend/
├── admin-panel/             # React admin dashboard
└── chat-widget/             # React embeddable chat widget
```

## Coding Conventions

### Python / FastAPI
- Use async/await throughout (asyncpg, async SQLAlchemy)
- Pydantic v2 for all schemas
- Type hints on all function signatures
- Services layer for business logic — routes should be thin
- Use FastAPI dependency injection for auth, DB sessions, brand context
- Custom exception classes with proper HTTP status codes
- Alembic for all schema changes — never raw SQL in code

### Database
- All tables have: `id` (UUID or serial), `created_at`, `updated_at`
- All brand-scoped tables have: `brand_id` (FK, indexed, NOT NULL)
- Use enums for: role, skin_type, concern, rule_type, channel, status
- JSON columns for flexible config (tone settings, rule configs)
- Encrypt at rest: secrets (AES-256), lead email/phone

### API Design
- RESTful, versioned: `/api/v1/...`
- Brand-scoped resources nested: `/api/v1/brands/{brand_id}/products`
- Consistent response format: `{ data, meta, errors }`
- Pagination: `?page=1&per_page=20`
- Filtering: `?channel=website&date_from=2026-01-01`
- RBAC enforced via FastAPI dependencies on every route

### Testing
- pytest + pytest-asyncio
- Test with at least 2 brands to verify isolation
- Mock external APIs (Claude, embeddings, S3) in unit tests
- Integration tests against real PostgreSQL (docker)

## Critical Business Rules
1. **Brand isolation is non-negotiable.** Every query filters by brand_id.
2. **No hallucination.** AI only uses retrieved context. No relevant context = fallback message.
3. **Compliance before delivery.** Every AI response passes through compliance filter.
4. **Moderation before AI.** Bad input blocked before Claude API call (saves tokens/cost).
5. **Async embeddings.** Never block admin CRUD waiting for embedding. Use Celery.
6. **Instant overrides.** Admin changes take effect immediately — cache invalidation required.
7. **Secrets never in plaintext.** Not in logs, not in admin UI, not in API responses.
8. **Session memory.** Don't re-ask questions. Don't re-recommend products. Build progressively.
9. **Graceful failure.** API down = fallback message. Embedding failed = retry queue. Never crash.
10. **Audit everything.** Every admin action logged with who/what/when/before/after.

## Phase 1 vs Phase 2
- **Phase 1** (current): Backend, RAG, admin panel, website chat widget, compliance, RBAC, leads
- **Phase 2** (future): WhatsApp, Instagram, live agent escalation, advanced analytics, CRM, A/B testing

## Ticket Workflow
- All work is tracked via GitHub Issues
- Labels: `phase-1`, `phase-2`, `backend`, `frontend`, `database`, `ai-engine`, `admin-panel`, `chat-widget`, `security`, `testing`, `devops`, `bug`, `enhancement`
- Issues are organized by development phase and priority
- Create a branch per ticket: `feature/{issue-number}-short-description`
- PR back to `main` when done

## What's Out of Scope
- Mobile native apps
- Voice chatbot
- Multi-language (English only in Phase 1)
- E-commerce checkout (link out only)
- Content writing (client provides)
- Marketing automation

---
> Source: [PruthvikAIRepo/Multi-Brand-AI-Chatbot-Platform](https://github.com/PruthvikAIRepo/Multi-Brand-AI-Chatbot-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
