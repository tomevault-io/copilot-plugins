## agent-workflow

> Agent workflow — planning, verification gates, tasks, lessons


# Agent Workflow

## Planning
Non-trivial (3+ steps or architecture): plan first; replan on failure. Verifiable checklist in `tasks/todo.md` before implementation.

## Subagents
Use for exploration and parallel research. One focused task per subagent.

## Verification (runtime code changes)
Run in order until green:
1. `make test` — full PHPUnit (`tests/Unit`, `tests/Feature`; SQLite in container)
2. `make pint-check` — on fail: `make pint-fix`, recheck diff
3. `make playwright` — E2E on port **8081** (Docker up: `make up`)

**Omit playwright**: `.md`-only or no PHP/TS/JS/navigable markup impact.
**PHP touched**: always steps 1–2 even if playwright omitted.

Never close without proof. Fix CI failures without hand-holding. Bug reports: diagnose logs/tests and fix.

## Tests required
Every new feature (macros, helpers, observers, services, renderers) needs ≥1 Unit or Feature test. Missing test = bug.

Third-party wrappers: assert delegate implements library interfaces (e.g. `ConfigurationAwareInterface`).

## Task hygiene
Plan → verify plan → track in `tasks/todo.md` → summarize → review section in todo. User correction → `tasks/lessons.md`.

## Code principles
Minimal diff, root cause, no drive-by changes. Non-trivial: prefer elegant solution; skip for obvious one-liners.

## PR
Relevant changes via PR. Merge: tests green, no duplication, acceptable perf, docs/security OK.

## Deploy / backfill dati
- **Entrypoint** (`docker/entrypoint.sh`): solo step idempotenti (migrate, optimize, sitemap, …).
- **One-shot** dopo cambio logica: **migration** Laravel (una volta per ambiente), mai artisan di backfill ripetuto a ogni deploy.
- Dettaglio: `docs/agent/architecture.md` · `docs/DEPLOY.md`.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
