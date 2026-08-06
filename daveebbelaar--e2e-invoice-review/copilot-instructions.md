## e2e-invoice-review

> Read `docs/client-brief.md`, `docs/architecture.md`, and `docs/build-along.md` before changing the project.

# Invoice Review agent instructions

Read `docs/client-brief.md`, `docs/architecture.md`, and `docs/build-along.md` before changing the project.

## Stack

- Backend: Python 3.12+, uv, FastAPI, Pydantic v2, SQLAlchemy 2, SQLite.
- Extraction: Azure AI Document Intelligence `prebuilt-invoice` and `prebuilt-receipt`.
- Independent review and categorization: Azure OpenAI Responses API with Entra authentication and strict structured output.
- VAT checks: local EU structure/checksum validation with `python-stdnum`; no live VIES claim.
- Frontend: Vite, React, TypeScript strict, Tailwind CSS, pnpm.
- Verification: Ruff for backend; TypeScript, ESLint, production build, explicit live evaluators, and a manual browser walkthrough for the complete flow.

## Boundaries

- Azure SDK types stop in `backend/app/providers/azure_document_intelligence.py`.
- OpenAI SDK types stop in the provider adapters under `backend/app/providers/`, including document review, GL suggestion, and correction-email drafting.
- The document reviewer receives the original PDF/PNG/JPEG and returns classification plus provider-independent structured fields. Document Intelligence remains primary; deterministic merging only fills its missing fields and exposes provenance.
- The GL categorizer receives normalized invoice fields only.
- The GL catalog and selection validation live in `backend/app/accounting/`; model output never becomes business policy.
- Business rules live in `backend/app/invoices/validation.py` and must be pure.
- HTTP concerns live in `routes.py`; orchestration lives in `service.py`; SQLite access lives in `repository.py`.
- Once those modules are introduced, settings are read only through `backend/app/config.py` and `frontend/src/lib/env.ts`.
- Do not add auth, queues, workers, deployment, batch processing, email ingestion/sending, or accounting integrations.
- Receipt processing uses the same normalized financial-document data and a separate deterministic policy. Live VIES registration lookup remains outside the build.

## Dependencies

- Never add a dependency without asking Dave first.
- Every dependency must earn its place. If only a small function is needed, propose implementing that function locally instead.
- Never run `uv add`, `pip install`, or `pnpm add` without explicit approval.
- When proposing a package, give the exact pinned version and one sentence explaining why it is better than local code.
- Pin direct dependencies exactly and commit `backend/uv.lock` and `frontend/pnpm-lock.yaml`.
- Keep `[tool.uv] add-bounds = "exact"` and `exclude-newer = "7 days"` in `backend/pyproject.toml`.
- Keep `savePrefix: ""`, `minimumReleaseAge: 10080`, and `minimumReleaseAgeStrict: true` in `frontend/pnpm-workspace.yaml`.
- Install with `uv sync --locked` and `pnpm install --frozen-lockfile`.
- Commands that must only run the existing backend environment use `uv run --locked --no-sync`.
- A cooldown exception requires explicit approval, a package/version-specific scope, and an adjacent explanation.

## Teaching guide

Update `docs/build-along.md` in the same commit as every working slice. Include the outcome, why, exact commands, observable result, and checkpoint.

## Verification policy

- Do not add automated test suites, `tests/` directories, or `*.test.*` files to this end-to-end teaching project.
- Keep verification proportional and demo-oriented: verify locked installs on the starter; as code is added, lint the backend, type-check/lint/build the frontend, exercise the fictional corpus evaluators when cloud usage is intended, and manually walk through the user story in the browser.
- Keep deterministic business rules and provider boundaries explicit and easy to inspect even though they are not backed by a committed unit-test suite.

## Secrets and data

Never commit `.env`, Azure keys, uploaded invoices, private documents, or SQLite databases. Generated samples must contain only fictional data.

---
> Source: [daveebbelaar/e2e-invoice-review](https://github.com/daveebbelaar/e2e-invoice-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
