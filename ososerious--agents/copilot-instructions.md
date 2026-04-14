## agents

> - __Scope__: Applies to the repository root `julep/`.


# Julep Project Rules

- __Scope__: Applies to the repository root `julep/`.
- __Audience__: Windsurf AI (Cascade) and human contributors.
- __Purpose__: Encode working agreements, code standards, and automation rules.

## Tech Context

- __Primary stacks__
  - Backend: Python 3.11+ (FastAPI/Django allowed), async-first, type-hinted.
  - Frontend: TypeScript (strict), React/Next.js acceptable.
  - Infra: Docker, Terraform, GitHub Actions.
  - Data: Postgres, Redis; Hasura metadata lives in `src/hasura/metadata/`.
- __Entry points__
  - Hasura metadata exports: `src/hasura/metadata/`.

## Code Quality Standards

- __General__
  - Favor simplicity (KISS) and avoid premature abstraction (Rule of Three).
  - Keep modules focused, functions small, side-effects explicit.
- __Python__
  - Type hints required. Use `mypy` (strict where feasible).
  - Format with `black`; lint with `ruff`.
  - Async for I/O; prefer `dataclasses`/Pydantic for models.
- __TypeScript__
  - `"strict": true`. Prefer functional, immutable patterns.
  - Runtime validation with `zod` when crossing trust boundaries.
  - ESLint + Prettier. Unit tests with Vitest/Jest.
- __Git__
  - Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `perf:`.
  - Keep PRs small and focused. Include context, screenshots for UI, and test notes.

## Architecture Principles

- Start simple monolith; modularize as needed. Event-driven when justified.
- Clean boundaries for domains. Avoid tight coupling to frameworks.
- Make illegal states unrepresentable via types and invariants.

## Security & Compliance

- No secrets in code. Use env vars/secret stores.
- Principle of least privilege for DB, cloud, CI.
- Validate all inputs at boundaries (API, jobs, queues).

## Testing Strategy

- Pyramid approach: unit >> integration >> e2e.
- Table-driven tests where helpful. Use fixtures/factories.
- Add regression tests for every bug fix.

## Observability

- Structured logs with correlation IDs.
- Metrics for SLIs/SLOs. Add tracing for critical flows.

## Task Master AI Conventions

- __Tasks file__: `.taskmaster/tasks/tasks.json` in repo root.
- __Tags__: Use tags to model work streams (e.g., `default`, `research`, `infra`).
- __Statuses__: `pending`, `in-progress`, `review`, `done`, `deferred`.
- __Subtasks__: Prefer 3–7 actionable subtasks per task.
- __Dependencies__: Avoid cycles; validate regularly.
- __PR linkage__: Reference task IDs in PR titles/descriptions (e.g., `TM-15`).

### Task Writing Guidelines

- Title: imperative, outcome-oriented.
- Description: user value, acceptance criteria.
- Details: implementation hints, interfaces, migrations.
- Test strategy: unit/integration scope, success criteria.

## Hasura Metadata

- Source of truth at `src/hasura/metadata/`.
- Always export deterministic metadata before commit.
- Review diffs for breaking permission/role changes.

## Review Checklist (PR author)

- __Builds__ locally and in CI.
- __Tests__ added/updated; pass locally.
- __Types__ clean (TS/py type checks).
- __Security__ reviewed (secrets/PII, perms).
- __Docs__/README updated if behavior changes.

## AI Collaboration (Windsurf/Cascade)

- Prefer proposing small, focused changes with context paths, e.g., `src/foo/bar.ts`.
- When editing files, include minimal, precise diffs and explain reasoning succinctly.
- If uncertain, ask for missing context rather than guessing APIs.
- Do not introduce new services/dependencies without explicit approval.

## Repository Hygiene

- Keep dependencies minimal and pinned. Remove unused packages.
- Add READMEs to new modules explaining responsibilities and boundaries.
- Name files and symbols descriptively; avoid abbreviations.

## Definition of Done

- Code merged to default branch.
- CI green; no critical lints.
- Tasks updated to `done` with notes and links to PR.
- Rollback plan documented for risky changes.

---

# Quick References

- Hasura metadata dir: `src/hasura/metadata/`
- Common statuses: `pending`, `in-progress`, `review`, `done`, `deferred`
- Commit style: Conventional Commits

EOF

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OSOSerious) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
