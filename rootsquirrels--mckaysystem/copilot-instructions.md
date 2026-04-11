## mckaysystem

> - Postgres is the **source of truth**.

# AGENTS.md

## Core principles

- Postgres is the **source of truth**.
- Everything is scoped by `tenant_id` + `workspace`.
- System must remain:
  - deterministic
  - idempotent
  - multi-tenant safe
  - reproducible in production

---

# Database rules (strict)

- All queries MUST include `tenant_id` AND `workspace`.
- Never modify a primary key without a migration.
- Never add/remove a column without a migration plan.
- Never perform silent DDL in runtime code.
- Lifecycle precedence is defined ONLY in `finding_current` view.

Lifecycle priority:

Do not reimplement lifecycle logic in Python.

---

# Ingestion rules

- Ingestion must be idempotent per `run_id`.
- Use `INSERT ... ON CONFLICT`.
- Update `runs` table last.
- No partial inconsistent commits.
- Code must remain compatible with future schema changes and API versions.


---

# Checker rules

- Deterministic output.
- Stable fingerprint.
- No randomness.
- No timestamp inside fingerprint.
- Must not crash on malformed AWS data.
- Must handle empty inputs.
- Must include unit tests.

---

# API rules

- Only query `finding_current` for findings.
- No `SELECT *`.
- Always filter by tenant/workspace.
- Avoid full table scans.
- Avoid N+1 queries.
- Use indexed columns in filters.

---

# Code quality (mandatory)

- Code must pass `pylint` with no new warnings.
- Do not disable pylint rules without strong justification.
- No broad `except Exception` unless strictly required.
- No unused imports.
- No dead code.
- Functions must have clear responsibilities.
- Prefer explicit typing.
- Avoid overly complex functions (refactor if needed).
- No hardcoded values.
- Write docstring for all functions.
- Use 'mypy' for type checking.
- use 'ruff' for linting.
- No `print()` statements in production code. Use logging.

Before committing:

Both pylint and tests must pass.

---

# Testing requirements

Every change must:

- Be deterministic.
- Be idempotent.
- Preserve tenant isolation.
- Not break empty DB behavior.
- Include/update tests if behavior changes.

Do not merge code that reduces test coverage.

---

# Forbidden changes

- Removing tenant/workspace scoping.
- Changing fingerprint format silently.
- Moving lifecycle logic outside DB.
- Adding hidden state.
- Introducing non-determinism.
- Runtime schema mutation without migration.

---

# Deployment safety

- DB schema must be version-aligned with code.
- Code must fail fast if schema mismatch is detected.
- No production-only logic branches.
- No environment-specific hacks.

---

This file defines non-negotiable production constraints.
If unsure, choose:
- determinism
- isolation
- explicitness
- migration-first approach

---

## Documentation Standards

- All public functions MUST have docstrings following Google style.
- Include: Purpose, Args, Returns, Raises, and Examples where helpful.
- Module-level docstrings should describe the module's purpose and key entities.
- README files are required for major components (apps/, services/, checks/).

## Error Handling

- Use specific exception types from `contracts/exceptions.py` when available.
- Always log errors before re-raising with context.
- API endpoints must return proper HTTP status codes (400 for bad request, 404 for not found, 500 for server errors).
- Validate all input payloads using `contracts/schema.py` validators.

## Logging

- Use `infra/logging_config.py` to configure logging.
- Use structured logging with context: `logger.info("message", extra={"key": "value"})`.
- Log at appropriate levels: ERROR for failures, INFO for significant operations, DEBUG for details.
- Never log sensitive data (credentials, PII, tenant data).

## Configuration

- All configuration MUST use `infra/config.py` or environment variables.
- Never hardcode values - use config files or env vars.
- Secrets must come from environment, never from config files.

## Database Access

- Use connection pooling via `apps/backend/db.py`.
- Always use parameterized queries - never string concatenation.
- Close connections in finally blocks or use context managers.
- Queries must include tenant_id and workspace filters.

## Testing Patterns

- Tests go in `tests/` directory, mirroring source structure.
- Use pytest fixtures from `tests/conftest.py`.
- Mock external services (AWS, Postgres) in tests.
- Each check must have corresponding tests in `tests/checks/`.

## API Design

- All endpoints must be versioned under `/api/v1/`.
- Use consistent response format from `apps/flask_api/utils/responses.py`.
- Query parameters must be validated using `apps/flask_api/utils/params.py`.
- All endpoints require tenant_id and workspace authentication headers.

## File Organization

- Business logic in `services/` or `contracts/`.
- API routes in `apps/flask_api/blueprints/`.
- Background workers in `apps/worker/`.
- Database migrations in `migrations/`.
- Shared utilities in `infra/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RootSquirrels)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RootSquirrels)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
