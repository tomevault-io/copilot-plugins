## fastapi-saas-template

> Execution contract for AI coding agents. Enforce architecture, constraints, and patterns. Prefer consistency over creativity.

# AGENTS.md

## Purpose

Execution contract for AI coding agents. Enforce architecture, constraints, and patterns. Prefer consistency over creativity.

## Instruction Priority

1. User task.
2. This file.
3. Existing repository patterns.
4. README.
5. Framework defaults.

## Before starting work

Codex/AI agents must:

1. Confirm current working directory and branch.
2. Read `AGENTS.md`.
3. Read `SESSION_NOTES.md` if present.
4. Read relevant docs from `backend/docs`.
5. Run `git status`.
6. Report the plan before editing.

## Structure

The backend is a modular monolith with explicit layers:

- `api`
- `services`
- `repositories`
- `schemas`
- `models`
- `core`

Domain modules live under:

```text
backend/app/<domain>/
  api/
  services/
  repositories/
  schemas/
  models/
```

Router registration entry point:

```text
backend/app/api/master_router.py
```

Expected flow:

`HTTP -> API -> Service -> Repository -> Service -> Response`

## Router contract

- Domain router files live in `backend/app/<domain>/api/*.py`.
- One file defines one router.
- Each router must define tags.
- Route paths must be defined inside domain router files.
- Routers should be attached in deterministic order with numbered comments when practical.
- `backend/app/api/master_router.py` is the only domain-router registration point.
- All API routes are attached to `v1_router`.
- Version prefix `/api/v1` is applied centrally in `master_router.py` through settings.
- `main.py` must not register domain routers directly.
- Do not pass extra prefixes in `include_router`, except the central version prefix.

## Layer responsibilities

- API handlers must stay thin.
- API must not contain business logic.
- Services contain business logic, orchestration, and authorization decisions.
- Repositories handle database access only.
- API layer must not access the database directly.
- Do not use raw SQL outside repositories unless explicitly justified.
- Use FastAPI dependency injection with `Depends`; avoid hidden globals.
- Use async only for database/external I/O; pure CPU logic should be sync.

## Persistence ownership

- Domain repositories own persistence for their aggregate tables.
- Platform services may orchestrate privileged workflows by calling domain repositories and platform-owned repositories, but must not duplicate basic persistence access for domain-owned tables.
- Platform repositories are allowed only for platform-owned tables, such as `platform_staff`, or dedicated platform read models/reporting queries that intentionally span multiple aggregates.
- Ownership mapping:
  - `users` table -> `UserRepository`
  - `organisations` table -> `OrganisationRepository`
  - `memberships` table -> `MembershipRepository`
  - `platform_staff` table -> `PlatformStaffRepository`
- Platform services own orchestration, permissions, audit event creation, conflict/not-found mapping, and state-transition decisions; they must not build SQLAlchemy queries for domain-owned aggregate tables.
- Platform organisation visibility is explicit: platform admin endpoints may include soft-deleted organisations for operational, audit, support, compliance, or recovery workflows; tenant-facing organisation endpoints must exclude soft-deleted organisations by default. `OrganisationRepository.get_by_id(..., include_deleted=False)` is the safe default, and platform services must pass `include_deleted=True` when they intentionally need deleted-organisation visibility.

## Transaction ownership

- Repositories may use `flush()` and `refresh()`, but must not call `commit()` or `rollback()`.
- Application services should not commit by default. Services orchestrate business rules, repository calls, and audit writes inside a transaction provided by the caller.
- Write API dependencies own transaction boundaries using `async with session.begin()` after authentication and rate limiting have completed.
- Read endpoints should use the lazy request-scoped session and should not open explicit transactions unless consistency requirements justify it.
- CLI commands and background workers must create their own explicit transaction boundary.
- Do not add global transaction middleware. It can start database work too early, weaken early auth/rate-limit short-circuiting, and make transaction scope less visible.

## API response contract

- Single resource: clean REST response.
- Collections: `{ "data": [], "meta": {}, "links": {} }`.
- Errors: Problem Details style with `application/problem+json`.
- Operational endpoints may return endpoint-specific payloads.

## Error handling contract

- API layer must not format business errors manually.
- API layer must not use `try`/`except` for business-flow errors.
- Services raise application/domain exceptions.
- Global FastAPI handlers format Problem Details responses.
- Do not leak internals, stack traces, tokens, secrets, or raw sensitive data.
- Token/credential-like flows must not reveal whether a token exists, expired, was revoked, or was already used; normalise external error responses.

## Security and auth

- JWT authentication uses Keycloak as identity provider.
- Authentication and authorization are separate concerns.
- Tenant authorization is resolved from local database memberships.
- Platform authorization is resolved from `platform_staff`.
- Permission logic belongs in services/dependencies, not arbitrary API code.
- Platform write endpoints must use the platform write rate limiting dependency/policy; do not add new platform write endpoints without rate limiting.
- Limited platform audit permissions must never expose raw metadata, IP address, user-agent, free-text reason, or direct actor identifiers.
- Do not trust client-provided identifiers, roles, or permissions.
- Do not implement local password login unless explicitly requested.

## Dependency management

- Python dependency management uses `uv`.
- The repository pins the local Python version in `.python-version`.
- Runtime dependencies live in `backend/pyproject.toml` under `[project.dependencies]`.
- Development dependencies live in `backend/pyproject.toml` under `[dependency-groups].dev`.
- `backend/uv.lock` is the only dependency lock source.
- Do not use Poetry.
- Do not use `pip-tools`.
- Do not recreate `requirements.txt` or `requirements-dev.txt`.
- Use Taskfile commands or `uv run` for local checks.
- Use `uv lock --check` to verify the lockfile is in sync.
- Use `uv sync --group dev` for local development dependencies.
- Use `uv sync --frozen --no-dev --no-editable` for production/runtime Docker installs.

## Logging

- Use structured JSON logging when configured.
- Never log passwords, tokens, API keys, or raw personal data such as email/IP.
- Logs must redact tokens, passwords, secrets, cookies, authorization headers, and API keys across common key variants.
- Mask or hash identifiers when needed.

## Configuration

- Use environment-based configuration only.
- Do not hardcode secrets, credentials, or deployment URLs.
- Keep local defaults safe for development.

## Testing

- Tests live under `backend/tests`.
- Lightweight tests are the default and normally do not need an execution marker.
- Explicit execution markers are reserved for integration, e2e, container, slow, contract, and external_db tests.
- Cross-cutting risk markers are limited to security, auth, authz, and privacy.
- Test business logic in services.
- Mock external dependencies in lightweight tests.
- Cover API behaviour with integration/e2e tests.

Before running pytest in a fresh local environment, install development dependencies from the lockfile:

```bash
cd backend
uv sync --group dev
```

Preferred repository-root commands:

```bash
task lint
task test:lightweight
task test:safe
task test:security
task test:auth
task test:authz
task test:privacy
task test:contracts
task ci
```

Direct backend commands must use `uv run`:

```bash
cd backend
uv run pytest -q -m "not external_db"
uv run ruff check .
uv run ruff format --check .
```

Prefer `task test:safe` or `uv run pytest -q -m "not external_db"` for broad safe checks.

Security regression checks can be selected with `task test:security` or:

```bash
uv run pytest -q -m "security and not external_db"
```

Focused security slices include `auth`, `authz`, and `privacy`. Focused runs are for local/manual diagnosis, not mandatory duplicate CI gates.

Use folders for domain/subsystem-specific runs instead of legacy micro-markers:

```bash
uv run pytest -q tests/rate_limit
uv run pytest -q tests/audit
uv run pytest -q tests/logging
uv run pytest -q tests/secrets
```

Do not add legacy micro-markers such as `unit`, `rate_limit`, `audit`, `cors`, `bola`, `logging_security`, or `secrets` to new tests.

Use this command as a lightweight marker-registration sanity check when updating security markers:

```bash
uv run pytest -q -m "security and not external_db" --collect-only
```

Documentation-only changes should run grep/link sanity checks and at least:

```bash
task lint
```

## CI

GitHub Actions runs the backend quality gate on pull requests and pushes to `main`.

The CI workflow must use:

- `.python-version`;
- `uv lock --check`;
- `uv sync --frozen --group dev`;
- Ruff format and lint checks;
- one broad non-external-db pytest run: `uv run --frozen pytest -q -m "not external_db"`;
- a branch-protection-safe aggregate CI status job.

Local equivalent:

```bash
task ci
```

## Change rules

- Read existing code before changing behaviour.
- Follow existing project patterns.
- Make the minimal necessary changes.
- Update tests and docs when behaviour changes.
- Do not change backend code for documentation-only tasks.
- Do not commit or push without explicit instruction from the controlling task.

## Forbidden

- Business logic in API handlers.
- Database access outside repositories.
- Logging sensitive data.
- Hardcoded secrets.
- Unnecessary new frameworks.
- Over-abstraction.
- Catch-all `utils`/`helpers` modules without narrow scope.
- Exposing ORM models directly as API responses.
- Unjustified try/except blocks around imports. Optional dependency/version compatibility fallbacks are allowed only when documented and tested.
- Reintroducing `pip-tools`, Poetry, `requirements.txt`, or `requirements-dev.txt` as dependency sources.

## Source of truth

1. Code is primary source of truth.
2. `AGENTS.md` controls AI-agent workflow.
3. `backend/docs/architecture.md` controls architecture docs.
4. `backend/docs/current-state.md` controls current status.
5. `SESSION_NOTES.md` controls live handoff state.
6. Feature-specific docs control details only for their area.

---
> Source: [AlexTymosh/fastapi-saas-template](https://github.com/AlexTymosh/fastapi-saas-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
