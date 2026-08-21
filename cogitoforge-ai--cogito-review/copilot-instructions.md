## cogito-review

> Instructions for AI coding agents working on this repository.

# AGENTS.md

Instructions for AI coding agents working on this repository.

## Naming

- Use `Cogito Review` as the product name in prose.
- Use `review`, `finding`, `repository integration`, `LLM provider`, `agent callback`, and `runtime provider` consistently with the existing docs.
- Keep literal code tokens exactly as they appear in source code, CLI commands, environment variables, API paths, and generated artifacts.
- Do not translate source-code identifiers, API names, or file names.

## Communication

- All source code, code comments, commit messages, and technical artifacts must use English.
- All chat messages to the user must use Vietnamese.
- Keep communication concise, direct, and action-oriented.
- When presenting plans, assumptions, risks, or implementation status, make them explicit so the user can review quickly.
- Do not use emoji in source code. Avoid overusing emoji in chat.

## Workflow

- Follow the repository workflow and AI contribution policy in [AI_POLICY.md](AI_POLICY.md).
- Do not duplicate or replace that workflow with ad hoc steps.
- If this file conflicts with `AI_POLICY.md`, surface the conflict to the user instead of guessing.

## Environment Setup

- Use the repository tooling directly. The standard local setup is `cp .env.example .env` followed by `make dev`.
- Install developer dependencies with `make install`.
- Use `uv` for Python package management and Node.js 22+ for frontend tooling.
- Prefer repository commands such as `make lint`, `make test`, and `make openapi` over custom one-off command chains.
- Use Docker Compose for stack lifecycle and integration-style local validation.
- Do not commit `.env`, tokens, generated secrets, or local machine credentials.

## Commands

- **Start the development stack:** `make dev`
- **Start the local prod-like stack from a repo checkout:** `make prod`
- **Stop the local prod-like stack:** `make prod-down`
- **Run database migrations:** `make migrate`
- **Roll back the latest database migration:** `make migrate-down`
- **Build the local agent image:** `make build-agent`
- **Render backend OpenCode config on the host:** `make render-opencode-config`
- **Regenerate backend OpenAPI and frontend API types:** `make openapi`
- **Run repository lint and frontend type checks:** `make lint`
- **Run the main test suite:** `make test`
- **Install dependencies and pre-commit hooks:** `make install`
- **Run backend integration tests:** `cd backend && uv run pytest -m integration`
- **Run backend unit tests only:** `cd backend && uv run pytest -m "not integration"`
- **Run shared tests only:** `cd shared && uv run pytest`
- **Run agent tests only:** `cd agent && uv run pytest`
- **Run frontend checks only:** `cd frontend && yarn test`

Run the smallest reliable validation scope first, then expand when the change crosses boundaries.

## Repository Structure

This repository is a monorepo with multiple runtime boundaries.

- `shared/` - cross-cutting Python package used by backend and agent
- `backend/` - FastAPI API, services, repositories, schemas, auth, worker entrypoints, and migrations
- `agent/` - isolated review runner, MCP server, toolbase integrations, and container entrypoints
- `frontend/` - React SPA with TanStack Router, TanStack Query, and generated OpenAPI types
- `operator/` - Kubernetes operator implementation
- `deploy/` - Docker, Helm, and raw Kubernetes deployment assets
- `docs/` - architecture, security, deployment, and product behavior documentation
- `website/` - documentation website sources
- `.github/workflows/` - CI, publish, operator, and website automation

Important generated or workflow-managed artifacts:

- `frontend/src/api/generated/schema.ts` - generated from `openapi.json`
- `openapi.json` - generated from the backend application
- Do not hand-edit generated artifacts when a generation workflow already exists.

## Architecture Boundaries

1. The backend is the source of truth for application state and persists data in PostgreSQL.
2. The worker consumes queued jobs and launches isolated review execution through runtime providers.
3. The agent performs review execution in isolation and reports results back through callback APIs.
4. The agent must remain database-independent. It must not read from or write to PostgreSQL directly.
5. Review execution boundaries matter: backend prepares work, worker launches work, agent executes work, backend persists results.
6. Shared packages define contracts and reusable logic, but should not blur the runtime boundary between backend and agent.
7. Frontend code should consume API contracts through generated types rather than handwritten response shapes.
8. Provider-specific logic should stay behind provider abstractions instead of leaking across services or routes.

For architecture details, read [docs/architecture-overview.md](docs/architecture-overview.md), [docs/worker.md](docs/worker.md), [docs/agent-executor.md](docs/agent-executor.md), and [docs/review-architecture.md](docs/review-architecture.md).

## Security Model

When reviewing code, changing authentication or authorization flows, or evaluating a possible vulnerability, keep the documented security model in mind.

- Webhook validation, callback authentication, RBAC, SSO, secrets handling, and runtime isolation are intentional security boundaries.
- The backend enforces authentication, authorization, and persistence boundaries.
- The worker is security-sensitive because it can launch isolated execution and may access the Docker socket in Docker mode.
- Agent containers report state through signed callbacks and should not gain direct database access.
- Settings, credentials, and provider configuration must not be hard-coded into source files.
- Treat changes to auth, RBAC, callback verification, runtime isolation, secret handling, and deployment defaults as high-risk.

Use [docs/security.md](docs/security.md), [docs/rbac.md](docs/rbac.md), [docs/sso-integration.md](docs/sso-integration.md), and [docs/deployment.md](docs/deployment.md) as the authoritative references.

## Coding Standards

- Follow existing patterns, conventions, layering, naming, dependency direction, and style already present in the codebase.
- Keep route handlers small, place business logic in services, and keep SQL in repositories.
- Use Pydantic schemas for API boundaries.
- Keep frontend data fetching in TanStack Query hooks rather than ad hoc `useEffect` plus `fetch` flows.
- Do not hand-edit generated files such as `frontend/src/api/generated/schema.ts` or router-generated artifacts.
- Use comments sparingly. Code should explain what happens; comments should explain why only when the reason is not obvious.
- Avoid unrelated refactors in task-scoped changes.
- Do not introduce new abstractions, internal frameworks, or generalized layers unless the task clearly requires them.
- For Python changes, run Ruff on the affected package before moving on.
- For frontend changes, keep TypeScript strictness intact and preserve the existing React and routing patterns.

### Backend async handlers

- Do not use sync FastAPI route handlers (`def`) for endpoints that read settings, touch PostgreSQL, or otherwise perform async I/O. Use `async def` with `await` and `Depends(get_conn)` (or an existing async service entrypoint) instead.
- Do not bridge sync and async with low-level helpers such as `asyncio.run()`, nested event loops, or sync wrappers that load DB state on cache miss. These patterns bind work to the wrong event loop when used with the shared `asyncpg` pool and can cause pool corruption (`InterfaceError`, tasks attached to a different loop).
- Keep URL/settings helpers pure when possible: load organization settings once with `await ensure_organization_settings(conn)` and pass explicit values downstream. Do not hide async I/O behind sync APIs used from handlers or services.
- Standalone CLI entrypoints under `backend/app/cli/` may use `asyncio.run()` because they run in a separate process and do not share the API pool.

## Testing Standards

- Every behavior change should add or update tests at an appropriate level when practical.
- Prefer the smallest reliable validation first, then broaden coverage if the change crosses package or runtime boundaries.
- Use `make test` for the standard repository suite.
- Run `cd backend && uv run pytest -m integration` for database behavior, migrations, integration-heavy backend logic, or flows that depend on persisted state.
- Run `make openapi` whenever backend contract changes affect frontend-generated types.
- If a change affects Docker packaging or runtime boot behavior, validate the relevant image build path.
- If a change affects release or CI automation, review the affected workflow file and validate the changed logic as far as feasible.
- If tests cannot be run, say so explicitly in the final report and explain what remains unverified.

## Output Conventions

- Keep user-facing summaries short, explicit, and easy to scan.
- State assumptions, risks, and validation results clearly.
- When work produces scratch notes, reports, or generated review artifacts for local inspection, place them under `files/` if they should remain in the workspace.
- Do not leave temporary debug files, temporary scripts, or noisy logs in tracked locations.

## Commits and PRs

- If you are on `main`, create a topic branch before making changes.
- Start feature and fix branches from the latest `main`.
- Open pull requests against `main`.
- Always ask the user before creating a PR.
- Keep commit messages concise, imperative, and focused on why the change matters.
- Before opening a PR, ensure the relevant validation has run: typically `make lint`, `make test`, and `make openapi` if contracts changed.
- If a change adds migrations, ensure they apply cleanly and call out operational impact in the PR summary.
- If a change affects docs, propose the doc updates to the user before editing documentation.
- Do not add yourself as a co-author in commit metadata.

## Boundaries

- **Ask first**
  - Business logic, schema, API contract, authentication, authorization, background jobs, integrations, or production data changes with unclear requirements.
  - Large cross-package refactors.
  - New dependencies with broad impact.
  - Destructive or difficult-to-reverse data changes.
  - Architecture changes that cross backend, worker, agent, and runtime boundaries.
- **Never**
  - Commit secrets, credentials, or tokens.
  - Edit generated files by hand when a generation workflow exists.
  - Redesign the architecture inside an unrelated task.
  - Use destructive git operations unless explicitly requested.
  - Expand scope beyond the user request without approval.

## References

- [README.md](README.md)
- [AI_POLICY.md](AI_POLICY.md)
- [docs/development.md](docs/development.md)
- [docs/architecture-overview.md](docs/architecture-overview.md)
- [docs/backend.md](docs/backend.md)
- [docs/frontend.md](docs/frontend.md)
- [docs/worker.md](docs/worker.md)
- [docs/agent-executor.md](docs/agent-executor.md)
- [docs/review-architecture.md](docs/review-architecture.md)
- [docs/security.md](docs/security.md)
- [docs/deployment.md](docs/deployment.md)
- [docs/observability.md](docs/observability.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)

---
> Source: [CogitoForge-AI/cogito-review](https://github.com/CogitoForge-AI/cogito-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
