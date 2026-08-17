## agentic-platform-baseline

> This repository is a reusable technical foundation for multi-tenant agentic products.

# Agentic Platform Baseline

## Purpose

This repository is a reusable technical foundation for multi-tenant agentic products.
It provides secure defaults and explicit extension boundaries without defining the
business rules, workflows, copy, or data semantics of a derived vertical.

## Source of truth

1. Approved requirements and acceptance criteria under `specs/`.
2. Architecture decisions under `docs/adrs/`.
3. Versioned contracts under `gcp/packages/contracts/`.
4. Implementation and automated tests.

Do not change behavior or acceptance criteria merely to simplify implementation. Record
material architecture or security decisions as ADRs.

## Architecture boundaries

- `frontend`: Next.js experience and App Hosting configuration; never an authorization
  boundary.
- `gcp/services`: cohesive Cloud Run HTTP services.
- `gcp/functions`: narrow, idempotent HTTP or CloudEvent adapters.
- `gcp/jobs`: finite migrations and isolated execution tasks.
- `gcp/packages/contracts`: public request, response, and event contracts.
- `gcp/firebase`: Firebase rules, indexes, tests, and constrained BaaS policy.
- `gcp/infrastructure`: local and cloud deployment definitions.
- `specs`: requirements, design, acceptance criteria, tests, and threat analysis.

Dependencies point inward. Domain modules never depend on FastAPI, SQLAlchemy, Firebase,
or provider SDKs. Customer-code execution never shares the control-plane process,
filesystem, credentials, network policy, or production identity.

## Template rules

- A derived project owns its product name, vertical domain, infrastructure decisions,
  data classification, and deployment approvals.
- Run `scripts/bootstrap_template.py` once before the first project commit.
- Never put credentials, real cloud project IDs, private endpoints, or production data in
  bootstrap configuration.
- AgentOps and MLOps are deferred extension phases. Do not present their contracts or
  runtime as implemented until the applicable specification is approved and tested.
- New deployables require evidence of independent scaling, security, lifecycle,
  ownership, or failure isolation.

## Canonical commands

Run from the repository root:

```text
pnpm install --frozen-lockfile
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test:coverage
pnpm build
uv sync --all-packages --all-extras
uv run ruff check .
uv run ruff format --check .
uv run mypy gcp/services/control-plane/src scripts
uv run pytest
uv run python scripts/validate_contracts.py
uv run python scripts/validate_architecture.py
uv run python -m scripts.validate_repository_hygiene
pnpm test:firebase
pnpm test:apphosting
docker compose config --quiet
```

On Windows, `scripts/validate.ps1` executes the broad local quality gate.

## Engineering rules

- Python 3.12+, strict mypy, Ruff, pytest, and branch coverage.
- Strict TypeScript, ESLint, Vitest, semantic HTML, responsive behavior, and keyboard
  accessibility.
- New or modified business logic requires 100% statement and branch coverage.
- Tenant-owned records and queries require trusted tenant context and forced RLS.
- External input is validated; provider output is normalized at its boundary.
- Outbound calls have explicit timeouts and retries are bounded and safe.
- Logs are structured, correlated, and exclude credentials or sensitive payloads.
- Database changes require migrations, rollback notes, and isolation tests.
- Firestore client access is deny-by-default; server access uses least-privilege IAM.
- Repository hooks and CI reject sensitive paths and high-confidence secrets.
- Google Cloud CI/CD uses Workload Identity Federation; key files are prohibited.
- Production promotion always requires an explicit human gate.

## Definition of Done

A change is complete only when acceptance criteria trace to meaningful tests, relevant
format/lint/type/build/security checks pass, contracts and documentation are current,
deployment implications are explicit, and remaining risks are disclosed.

---
> Source: [DavidFerj/agentic-platform-baseline](https://github.com/DavidFerj/agentic-platform-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
