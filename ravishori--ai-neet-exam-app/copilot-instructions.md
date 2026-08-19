## ai-neet-exam-app

> Read this before touching any code in this repo. It exists so every session

# CLAUDE.md — Trinetra AI Learning OS (TALOS)

Read this before touching any code in this repo. It exists so every session
starts with the same frozen decisions instead of re-deriving them.

## What this is

An AI-first learning platform, NEET as the first product. Full brainstorm is
in `BRD.docx` (23k lines — treat as backlog/vision, not a build spec) and
`Trinetra AI Learning OS (TALOS).docx` (README-style snapshot). The actual
build target is the phased plan below, not the BRD's enterprise-scale vision
(280 tables, 12 AI agents, full knowledge graph — all deferred, see ADRs).

## Frozen decisions (do not re-litigate — see docs/decisions/ for the "why")

- **Architecture**: modular monolith. One FastAPI app, one Next.js app. No
  microservices, no separate admin frontend app.
- **Stack**: Next.js 15 + TS + Tailwind + shadcn/ui · FastAPI + SQLAlchemy 2.x
  (async) + Alembic + Pydantic v2 · PostgreSQL 17+ · Redis.
- **Auth**: custom JWT (access + rotating refresh tokens), Argon2 password
  hashing, HTTP-only cookies. Not Auth.js.
- **AI**: AI Gateway abstraction from day one, Claude as the only wired
  provider for now. Four agents in v1: Tutor, Question Generator, Study
  Planner, Evaluator. Nothing else (Mentor, Digital Twin, Diagram Agent,
  12-agent orchestrator) until v1 ships.
- **Content**: NCERT-aligned + originally authored content only. No
  ingestion of Aakash/Allen/PW/Unacademy material without explicit
  licensing. Content moves through the ECAEP workflow
  (`docs/architecture/ecaep.md`) — never a CRUD path that skips review.
- **Commerce**: Razorpay. **Hosting**: Coolify on a Hetzner VPS for MVP.
- **Multi-tenancy**: not in MVP. Reserve an `organizations` table; don't
  thread `tenant_id` through anything yet.
- **Naming**: always "Trinetra AI Learning OS (TALOS)", never "AI Learning
  OS" (an earlier working title that appears throughout the BRD).

## Repo conventions

- Backend modules live under `apps/backend/app/modules/<name>/` with
  identical internal shape: `api/ services/ repositories/ models/ schemas/
  tests/`. See `apps/backend/app/modules/identity/` once it exists as the
  template for every module after it.
- Every table: `id UUID PK`, `created_at/updated_at TIMESTAMPTZ`,
  `created_by/updated_by`, `deleted_at` (soft delete), `version INT`.
- PostgreSQL schemas, not everything in `public`: `identity`, `academic`,
  `cms`, `assessment`, `ai`, `analytics`, `commerce`, `system`.
- Alembic migrations are the only way schema changes happen. Never hand-edit
  a deployed schema.
- API responses follow one envelope: `{ success, data, meta, errors,
  traceId, timestamp }`.

## Where to look

- `docs/decisions/` — ADRs, one per frozen decision above.
- `docs/architecture/roadmap.md` — the sprint-by-sprint build plan (SP0
  Foundation → SP1 Identity → SP2 Academic → SP3 CMS+Questions → …).
- `docs/architecture/ecaep.md` — the content editorial workflow spec.

---
> Source: [ravishori/ai-neet-exam-app](https://github.com/ravishori/ai-neet-exam-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
