## opensplit

> - Backend: FastAPI + SQLAlchemy + PostgreSQL (Supabase). No Flask, no Django, no MongoDB.

# OpenSplit Rules

## Stack
- Backend: FastAPI + SQLAlchemy + PostgreSQL (Supabase). No Flask, no Django, no MongoDB.
- Frontend: Next.js 14 App Router + TypeScript + Tailwind. No Pages Router.
- Auth: JWT + Supabase Auth. Guest sessions use `is_guest: true` JWT claim.

## Architecture
- Routes → Services → DB. No business logic in routes. No direct DB calls in routes.
- Agent logic in `backend/agents/`. All LLM calls via `backend/agents/llm_client.py` only.
- Background jobs in `backend/tasks/` via Celery.

## Naming
- Python: `snake_case` files/functions, `PascalCase` classes, `UserModel` for DB models, `UserCreateSchema` for Pydantic.
- TypeScript: `PascalCase` components, `useHookName` hooks, `kebab-case.tsx` component files.

## Critical Rules
- **Money = integers only** (store in cents/smallest unit). Never floats.
- Currency stored as original. FX conversion at display time only. Rates cached 1hr in Redis.
- Every table: `id (UUID)`, `created_at`, `updated_at`, `deleted_at` (soft delete).
- All routes return: `{ "data": {}, "message": "success", "error": null }`
- Mobile-first. 360px min width. 48px min touch targets. Dark mode default.
- No `any` in TypeScript. No inline styles.
- Never call external APIs from frontend — always go through backend.
- No secrets in code. Use `.env` only.
- All DB migrations via Alembic. Never edit existing migrations.

## Brain-First AI Workflow
- For non-trivial changes, query Brain first before editing code:
	- `find-symbol`, `who-calls`, `trace-flow`, `impact-analysis`, `semantic-search`
- Apply edits only after Brain context is collected and cross-checked.
- Prefer `doctor`/`health` checks before and after larger refactors.
- Use `index --force` after parser/analysis upgrades so stale hashes do not hide new findings.

### Auto-activation trigger
- If the user prompt includes `BRAIN_AUTO`, Brain analysis is mandatory before editing.
- If the user prompt uses aliases like `use brain workflow`, `brain workflow`, `use brain-first`, or `run brain first`, treat it as `BRAIN_AUTO`.
- Run at minimum: `find-symbol` + `who-calls` + `impact-analysis` + `semantic-search`.
- Provide a short findings summary, then perform only scoped edits derived from those findings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SanoKhan22)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SanoKhan22)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
