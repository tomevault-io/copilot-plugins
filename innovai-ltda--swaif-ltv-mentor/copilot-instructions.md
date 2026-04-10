## swaif-ltv-mentor

> **Workspace Instructions File**

# AGENTS.md

**Workspace Instructions File**

This file is the authoritative workspace instructions for both AI agents and human contributors. It defines the rules, conventions, and guardrails for all code and documentation changes in this repository.

**Principle:** Link, don’t embed — always reference detailed documentation rather than duplicating content. See the "See also" section below for direct links to key docs.

## Startup Rule

Before making any code change, read these files first in this order:

1. `project-context.md`
2. `docs/architecture/platform_architecture_operational_model.md`
3. `docs/mvp-mentoria/frontend-integration-architecture.md`
4. `docs/mvp-mentoria/contracts-freeze-v1.md`
5. `docs/mvp-mentoria/frontend-deployment-readiness-checklist.md`


If the task touches only one area, still read `project-context.md` first, then only the most relevant docs.

---

## Mission

This repository is in **final stabilization / final adjustment mode**.

Your job is to:
- make the smallest safe change
- preserve established architecture and conventions
- avoid unrelated refactors
- keep changes production-focused
- favor high signal over broad exploration


Do not redesign the system unless explicitly asked.

---

## Mandatory Working Rules

- Read `project-context.md` before implementing.
- Follow the stricter interpretation when in doubt.
- Keep changes localized to the nearest existing module.
- Prefer extending existing code over creating parallel paths.
- Summarize the likely files to change before editing.
- Explain the smallest safe patch before applying it.
- After changes, run the smallest relevant validation first.
- Do not make speculative “improvements” outside the task.
- Do not re-open solved architecture decisions unless the task requires it.

---

## Frontend Guardrails

- Treat TypeScript strict mode as a hard constraint.
- Do not introduce `any` unless the boundary is explicitly unknown and unavoidable.
- Keep raw API payloads as `unknown` at the boundary and normalize through adapters.
- Do not put alias normalization or contract migration logic in React components.
- Reuse the existing `AppError` flow for network and HTTP failures.
- Do not read `import.meta.env` directly in feature code.
- Centralize environment access in `src/shared/config/env.ts`.
- Keep data fetching out of page JSX; use the established domain-hook/resource pattern.
- Reuse existing loading / error / empty state conventions.
- Keep routing centralized in `src/app/routes.tsx`.
- Preserve existing shell patterns such as `AdminShell`, `MentorShell`, and `StudentShell`.
- Keep feature code inside existing domain structure before adding new top-level namespaces.
- Preserve Portuguese user-facing copy where that convention already exists.

---

## Backend Guardrails

- Keep FastAPI route handlers thin.
- Put business logic in `backend/app/services`.
- Keep persistence concerns in repositories under `backend/app/storage`.
- Register endpoints through routers under `backend/app/api/routes`.
- Do not place endpoint logic in `backend/app/main.py`.
- Preserve the standardized API error envelope:
  `{ error: { status, code, message, details } }`
- Do not invent endpoint-specific error shapes.
- Keep request validation in schemas and shared API error utilities.
- For auth-sensitive logic, use existing security/service patterns rather than ad hoc parsing.

---

## Testing Rules

- Do not add or change behavior without coverage at the nearest relevant layer.
- Frontend tests belong in `frontend/src/test`.
- Backend tests belong under `backend/tests` with the existing scope split.
- For API changes, preserve and update tests for standardized error payloads.
- For service-rule changes, update service-layer tests.
- For repository behavior, prefer tests that exercise the JSON-backed storage layer.
- For frontend auth or routing changes, update the relevant smoke-style route tests.

---

## Critical Don’ts

- Do not bypass adapters by binding raw API payloads directly into React state.
- Do not spread env access, token storage logic, or fetch details across feature code.
- Do not put business rules in route files.
- Do not put persistence logic in service callers.
- Do not reintroduce demo credentials, preview sessions, localhost defaults, or demo copy into production paths.
- Do not silently change frozen contract expectations.
- Do not create broad utility abstractions when a small explicit function is enough.
- Do not refactor unrelated files during final adjustments.

---

## Response Pattern

For each task:

1. Briefly restate the requested change.
2. List the most likely files involved.
3. Describe the smallest safe patch.
4. Apply only the scoped change.
5. Run the smallest relevant validation.
6. Summarize exactly what changed, any risks, and any follow-up checks.

If the requested change appears to require broader architectural movement, stop and say so explicitly before making wide edits.

---

## Priority Order

When rules compete, use this order:

1. `project-context.md`
2. frozen contracts and architecture docs
3. existing code patterns
4. task-specific instructions from the user
5. generic best practices

Generic best practices must not override documented project conventions.

---

## Continuation Rule

This is an existing brownfield repository already under active development.

Do not behave as if this is a greenfield project.
Do not regenerate architecture from scratch.
Do not introduce new patterns unless the current task clearly requires them.
Resume from the existing codebase and existing conventions.

---

## See also

- [project-context.md](project-context.md)
- [Platform Architecture & Operational Model](docs/architecture/platform_architecture_operational_model.md)
- [Frontend Integration Architecture](docs/mvp-mentoria/frontend-integration-architecture.md)
- [Contracts Freeze v1](docs/mvp-mentoria/contracts-freeze-v1.md)
- [Frontend Deployment Readiness Checklist](docs/mvp-mentoria/frontend-deployment-readiness-checklist.md)
- [README.md](README.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/INNOVAI-LTDA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/INNOVAI-LTDA)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
