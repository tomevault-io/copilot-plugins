## genwolf-oss

> This document defines how automated agents and contributors should work

# AGENTS.md — genwolf-oss

This document defines how automated agents and contributors should work
with the **genwolf-oss** monorepo.

The goal is to ensure consistency, correctness, cost efficiency, and
maintainability of the system.

---

## Monorepo Structure

```
genwolf-oss/
  apps/web/       — Product application (Next.js, API routes, DB access)
  apps/worker/    — Backend workers (Python, LLM processing, enrichment)
  docs/           — Unified documentation (features, architecture, setup, operations)
  infra/          — Docker Compose for local dev (Postgres, RabbitMQ, web, worker)
  .env / .env.example — Shared environment config (root level)
```

## Documentation Rules (Strict)

- `/docs` is considered a first-class part of the codebase
- After **every implementation change**, agents MUST:
  - review relevant files in `/docs` and README.md
  - update them if behavior, assumptions, or data flow changed

- A task is NOT considered complete if:
  - implementation changed
  - but `/docs` no longer reflects reality

- If no documentation changes are required:
  - explicitly state this in the response

## Testing & Validation

After **every implementation change**:

- ensure the application builds
- ensure all existing tests pass
- add tests for new or changed behavior when applicable

An implementation is not considered complete if:

- existing tests are failing
- new behavior is untested without explicit justification

---

## Web Application (`apps/web/`)

The main product application responsible for:

- the user-facing dashboard (Next.js)
- backend API routes and server logic
- database schema and queries (Prisma)
- orchestration and consumption of data produced by the worker

This project defines the **product contract** of Genwolf.

### Auth

When creating routes intended for the worker:

- update `proxy.ts` accordingly
- use **service JWT authentication** (`Authorization: Bearer <JWT>`, not session-based auth)
- ensure traffic from workers is explicitly allowed

### Database

We use **Prisma**.

Relevant files:

- `apps/web/prisma.config.ts`
- `apps/web/prisma/schema.prisma`

### Cross-References

Agents working on `apps/web/` MAY inspect `apps/worker/` to understand:

- pipeline outputs
- metric semantics
- enrichment logic

For additional context, review `docs/architecture/system-overview.md`.

---

## Worker (`apps/worker/`)

A backend worker responsible for:

- extracting brand mentions from AI-generated answers
- classifying brand roles (own / competitor / authority / unknown)
- enriching brands with domains and metadata
- computing analytics signals (mentions, sentiment, trends)
- optimizing LLM usage (cost, caching, batching, fallbacks)

This project is **LLM-heavy and cost-sensitive**.

### Code Style & Architecture

- Python files should generally **not exceed ~700 lines**
  - If they do, introduce modules and refactor
- Prefer **small, composable functions**
- Avoid deeply nested logic
- Keep LLM-related logic isolated (prompts, retries, fallbacks)

### Testing

After **any implementation change**:

1. Activate the virtual environment:
   `source venv/bin/activate`

### Cross-References

Agents working on `apps/worker/` MAY inspect `apps/web/` to understand:

- API contracts
- data expectations
- frontend-driven constraints

---

End of document.

---
> Source: [OskarWieckowicz/genwolf-oss](https://github.com/OskarWieckowicz/genwolf-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
