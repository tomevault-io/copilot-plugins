## marketfinder-etl

> description: End-to-end engineering guidelines for MarketFinder-ETL (TypeScript-first, poly-repo with Python + Airflow)


---
description: End-to-end engineering guidelines for MarketFinder-ETL (TypeScript-first, poly-repo with Python + Airflow)
globs: |
  **/*.ts,**/*.tsx,**/*.js,**/*.jsx,
  scripts/**/*.ts,
  apps/**,api/**,
  dags/**/*.py,scripts/**/*.py
---

# General
1. **Strict TypeScript**  
   • `noImplicitAny`, `strictNullChecks`, `exactOptionalPropertyTypes` must stay enabled (see repo [tsconfig.json](cci:7://file:///c:/Workspace/Code/marketfinder_ETL/tsconfig.json:0:0-0:0)).  
   • Never suppress with `// @ts-ignore`—prefer explicit types or overloads.

2. **Module Resolution**  
   • Use absolute imports via `@/*` alias (configured in `tsconfig.paths`).  
   • Node ESM only; no `require()`.

3. **Environment & Secrets**  
   • All config in `.env` – access via the central `src/utils/env.ts` helper.  
   • Never log raw API keys; mask with `****` helper in logs.

4. **Logging / Monitoring**  
   • `pino` for Node logs, `console.log` is banned.  
   • Each ETL step MUST emit a `{ step, durationMs, records }` object.

# Directory conventions
| Path                         | Purpose                                               |
| ---------------------------- | ----------------------------------------------------- |
| `src/utils/`                 | Pure, re-usable helpers (no I/O side-effects).        |
| `scripts/production/`        | One-off batch jobs; use `ts-node` shebang.            |
| `apps/etl-service/src/api/`  | HTTP endpoints – must be split by resource.           |
| `convex/`                    | DB logic (see separate *convex_rules*).               |
| `dags/`                      | Airflow DAGs – one **daily** process per file.        |

# ETL-pipeline rules
1. **Idempotency** – every script accepts `--since <ISO>` to re-run incrementally.  
2. **Batch size** – default `1000`; override with `BUCKET_PROCESSING_BATCH_SIZE` env.  
3. **Error handling** – wrap external API calls in `retry(fn, { retries: 3, backoffMs: 500 })`.  
4. **DuckDB usage** – never `DROP TABLE`; use `CREATE OR REPLACE`.  
5. **ML layer** – model artifacts saved under `./models/<date>/`; never commit large weights.

# API / Service
• All routes under `/api/*` MUST return `{ success, data?, error? }` JSON.  
• Validate inputs with `zod`; respond `422` on failure.  
• Health probe lives at `/api/health` and must only check DB + external API credentials.

# Testing
• `vitest` for TypeScript, `pytest` for Python.  
• Minimum coverage: `70%` lines per package (enforced in CI).  
• Use `nock`/`responses` for API mocks—no live calls in unit tests.

# Lint / Format
• `eslint --max-warnings 0` passes in CI.  
• `prettier` auto-runs via Git pre-commit hook.

# Python specifics (dags/, scripts/)
1. **PEP 621** in [pyproject.toml](cci:7://file:///c:/Workspace/Code/marketfinder_ETL/pyproject.toml:0:0-0:0); dependencies locked via [poetry.lock](cci:7://file:///c:/Workspace/Code/marketfinder_ETL/poetry.lock:0:0-0:0).  
2. Black + isort mandatory (`black --check`, `isort --check`).  
3. Type-hint every function; enable `mypy --strict`.

# Security
• Rotate Kalshi / Polymarket tokens weekly – stored in Convex secret store.  
• LLM calls routed through central `src/utils/llmClient.ts`, which enforces rate-limit and audit logging.

# CI / CD
• PR branch must green-light: lint → unit test → type-check → build (`npm run build`) in < 10 min.  
• Main branch auto-deploys the ETL service Docker image to staging; prod requires manual approval.

---
Follow these guidelines when creating or modifying code to keep the project performant, secure, and consistent.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/algorhythmic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
