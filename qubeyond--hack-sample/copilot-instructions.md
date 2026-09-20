## hack-sample

> Hackathon starter: three independent services, connected only over HTTP — no shared code, each with its own deps/Dockerfile/.env.

# sample

Hackathon starter: three independent services, connected only over HTTP — no shared code, each with its own deps/Dockerfile/.env.

## Stack
- `backend/` — FastAPI + Pydantic + Postgres (SQLAlchemy), layered so dependencies point one way — framework/DB details never leak into business logic. Exact layering documented in `backend/CLAUDE.md`.
- `frontend/` — Vite + React + TypeScript, organized by feature rather than by file type, shadcn/ui + Tailwind v4 for UI. Documented in `frontend/CLAUDE.md`.
- `ml/` — FastAPI, no business-entity layer — it just loads a model and runs it. Documented in `ml/CLAUDE.md`.
- Package managers: `uv` (backend, ml), `npm` (frontend). Lint/format: `ruff` (backend+ml, shared config), `eslint`+`prettier` (frontend) — always run format and lint together, never separately (`make lint`).
- Ports: backend `8000`, frontend `5173`, ml `8100`. Postgres via docker-compose; sqlite fallback for backend without docker.

## Working order for a feature that spans services
1. Agree the HTTP contract first — method, path, request/response shape, error codes — before either side writes code against it. It belongs wherever the owning service already keeps its request/response types, never a separate shared doc.
2. Implement backend and/or ml behind that contract.
3. Build the frontend feature against it last — it's the consumer, not the source of truth.

When a contract needs to change, rewrite the schema. Don't bolt on optional fields to preserve compatibility for consumers that don't exist yet — there's no legacy to protect in a hackathon.

Not every feature touches all three services — skip what the task doesn't need.

## Rules
- Never import code across `backend/`, `frontend/`, `ml/` — HTTP only.
- Every endpoint returns a typed schema, including health checks — no raw dicts.
- No comments in code, except to explain a non-obvious constraint or workaround.
- MVP scope: don't add layers, abstractions, or dependencies the current task doesn't need.
- Before marking a task done, run the lint command for the service you touched.
- Each service owns its ignore rules and example env file — don't move service-specific state into the root ones. Root versions only hold what's actually root-level (docker-compose vars, editor junk).

## Skills
Situational — use when the trigger applies, don't force them elsewhere:
- `brainstorm` — before any creative work (new feature, component, behavior change). Explores intent and 2-3 approaches with trade-offs, gets design approval, then hands off to `task-breakdown`.
- `task-breakdown` — turn an approved design/spec into small, ordered, independently-testable tasks with exact file paths before touching code.
- `feature-interview` — the request is ambiguous or high-stakes and the user's exact mental model matters more than shipping any working version. Interview via text before writing code.
- `clean-architecture` — adding or reviewing a change that touches backend's layering (or any service that grows a similar entity/business layer). Checks dependency direction and flags unnecessary interfaces.
- `kent-beck-style` — reviewing code for smells, naming, duplication, or "is this over-engineered" — the sharper checklist version of the MVP-scope rule above.

Skills load at session start — after adding or editing one, restart the session before expecting it to show up.

## Commands
Run `make` from the repo root (that's where the Makefile lives).
- `make up` — full stack via docker, builds automatically
- `make backend-dev` / `make frontend-dev` / `make ml-dev` — run one service without docker, deps installed automatically
- `make lint` — format + lint all services
- `make hooks` — install pre-commit hooks (after `git init`)

---
> Source: [qubeyond/hack-sample](https://github.com/qubeyond/hack-sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
