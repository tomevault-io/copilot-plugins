## open-submission-approval-workflow

> Issues are tracked in GitHub Issues. External PRs are not part of the triage surface. See `docs/agents/issue-tracker.md`.

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues. External PRs are not part of the triage surface. See `docs/agents/issue-tracker.md`.

### Triage labels

Triage uses the default canonical labels: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, and `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

This repo uses a single shared domain context with one root `CONTEXT.md` and root `docs/adr/`. The codebase is a `pnpm` monorepo managed with Turborepo, with an AdonisJS backend in `apps/backend` and a React/Vite frontend in `apps/frontend`. See `docs/agents/domain.md`.

## Assessment guardrails

- This repository is for **Assignment B: Submission & Approval Workflow** from the Open Ownership full-stack assessment.
- Optimize for the rubric over feature breadth. In order: workflow correctness, server-side authorization, meaningful tests, clear assessor-facing documentation, then optional enhancements.
- Prefer shipping a smaller but finished slice over speculative breadth. Search, notifications, richer queue tooling, and attachment versioning are post-core enhancements unless they are nearly free.

## Monorepo rules

- `pnpm` is the canonical package manager. Do not add `npm`-first instructions unless they are a compatibility note.
- The repository root is the primary operator surface. Prefer committed root scripts over ad hoc command sequences in docs and agent output.
- `pnpm dev` from the repo root is the canonical local development entry point and maps to `turbo dev`.
- Local infrastructure uses `apps/backend/compose.yml` for PostgreSQL and Mailpit. The backend and frontend run locally outside Docker during development.

## Architecture rules

- Treat `Application` as the core business record. Use the glossary in `CONTEXT.md` and do not drift to alternate terms like "submission" or "request" for the record itself.
- Keep one shared domain glossary for the product. This repo is a technical monorepo, not a multi-context domain model.
- Model workflow transitions as explicit resources around `Application`, not as a generic status patch.
- Keep applicant and reviewer route surfaces explicit and separate when their behavior differs.
- Every workflow transition must be atomic: update application state and write the audit log in the same database transaction.
- The audit log records status transitions, not generic draft edits.
- Fixed option lists belong to backend code and are exposed through API endpoints. Database columns should remain portable strings rather than database enums.
- List endpoints should include pagination and a sensible default ordering from the moment they are introduced.
- Route, controller, service, test, and UI names should mirror the workflow vocabulary already captured in `CONTEXT.md`.

## Implementation order

- Build workflow features backend-first: schema, domain/service logic, routes/controllers, tests, then frontend wiring.
- Keep controllers thin. Put workflow rules in dedicated services and keep authorization checks at the entry points, adjacent to workflow invariants.
- Use Problem Details style error envelopes from `apps/backend/app/exceptions/handler.ts` for validation, authorization, not-found, and illegal-transition responses.
- Prefer backend-provided validation messages as the source of truth. If wording needs refinement, customize it in backend validators rather than inventing a second frontend validation language.

## Documentation rules

- The root `README.md` is the assessment deliverable and the primary source of truth. App-level READMEs are supplementary workspace notes only.
- The root README should be optimized for assessor handoff first and should explicitly frame the repo as Assignment B.
- Keep the README narrative, but use rubric-aligned section labels so an assessor can find setup, workflow rules, data model, authorization, testing, trade-offs, deployment, and AI usage quickly.
- Include a small architecture diagram in the root README.
- AI usage disclosure in the final README must be specific and auditable: tool name, how it was used, and what was manually verified.
- Major architectural choices should stay aligned across code, ADRs, and the root README. If a decision materially shapes the system, document it where an assessor will see it.

## Deployment rules

- Production targets Sevalla with the backend on an `api` subdomain and the frontend on the main app domain.
- Sevalla redirects should proxy `/api` requests from the frontend app to the backend service.
- Session auth is the default production auth model. The frontend talks to the backend with cookie-based sessions, and the backend must be configured with explicit production CORS for the frontend origin.
- The backend is deployed as a containerized AdonisJS app. The frontend is deployed as a static site build. PostgreSQL is managed.

## Commit and PR rules

- Use Conventional Commits for every commit in the form `<type>(<scope>): <short imperative summary>`.
- Prefer explicit scopes tied to the changed area, such as `workflow`, `auth`, `reviewer`, `applicant`, `docs`, `deployment`, or `migrations`.
- If changes span unrelated concerns, split them into separate scoped commits.
- Every commit message must include a body with specific `-` bullet points covering why the change was needed, what changed, and any relevant impact, risk, migration, or test notes.
- When making commits from the shell, use a heredoc so multiline commit bodies are preserved correctly.
- Before creating or materially updating a PR, read the repository PR template and use its exact structure.
- PR titles should also follow Conventional Commits.
- One PR may contain multiple closely related vertical slices from the same branch when that produces a more coherent review unit. Avoid creating extra branches when the slices are tightly coupled.
- Even when one PR contains multiple slices, keep the related issues and test coverage explicit so the reviewer can still assess each slice clearly.

## Current ADRs

- `docs/adr/0001-same-origin-session-auth-on-sevalla.md`
- `docs/adr/0002-explicit-transition-resources-for-application-workflow.md`

---
> Source: [stctheproducer/open-submission-approval-workflow](https://github.com/stctheproducer/open-submission-approval-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
