## velonus

> Velonus is an AI-native application security platform for developers. Instead of

# Velonus

Velonus is an AI-native application security platform for developers. Instead of
generating hundreds of noisy alerts, it runs static analysis (secrets, Bandit,
Semgrep, pip-audit, Safety), normalizes and deduplicates the findings, uses AI to
triage them by actual exploitability, explains why each one matters in plain
English, generates a secure code fix, and — optionally — opens a one-click GitHub PR
with that fix plus an AI-generated regression test. The startup pivoted from code
encryption/runtime hardening to this developer-first AppSec automation wedge; the
core insight is "developers want fixes, not more alerts." Target market for the MVP
is Python developers, AI startups, and small teams without dedicated AppSec.

**The single source of truth for this project is
[`.github/copilot-instructions.md`](.github/copilot-instructions.md).** Read it in
full before starting any task — it holds the full product overview, the phased
roadmap (Phase 0 → Phase 12) with an inline progress tracker, the locked tech stack,
AI-engine rules that must never be violated, the API design, the database schema
summary, and hard DO/DON'T engineering constraints. This file is intentionally short
and does not duplicate it.

Supporting docs: [`docs/architecture.md`](docs/architecture.md),
[`docs/api.md`](docs/api.md), [`docs/database-schema.md`](docs/database-schema.md)
(see [`docs/schema-registry.md`](docs/schema-registry.md) for the as-built schema —
the two have known drift, tracked in
[`docs/open-questions.md`](docs/open-questions.md)),
[`docs/ai-prompts.md`](docs/ai-prompts.md), [`docs/decisions/`](docs/decisions/)
(ADR log).

**Current phase:** Phase 7 — Gap Closure, Robustness & Differentiation Sprint (in
progress). Phases 0–6 are done. Within Phase 7: 5A/5B/5C are complete, 5E is mostly
complete (7/8). 5D (VS Code extension), 5F (suppression rules engine), 5G (scan
comparison/trend analytics), 5H (SSE real-time progress), and 5I (re-triage/SBOM/
CI-template/Slack/SLA) are genuinely unstarted.

**Tech stack (locked — do not change without explicit approval):** Python + FastAPI
(`apps/api`) + Typer CLI (`apps/cli`, published as `velonus`), uv (not pip/poetry),
ruff + mypy strict (not black/isort/flake8), SQLAlchemy async ORM + asyncpg +
Alembic, ARQ (not Celery), Next.js 14 App Router + Tailwind + shadcn/ui
(`apps/web`), Clerk (not Auth0/custom auth), Docker Compose (Postgres 16 + Redis,
no pgvector), Railway. Static analysis tools (Bandit/Semgrep/pip-audit/Safety/
detect-secrets) are wrapped, never reimplemented. AI layer is Anthropic Claude only
(triage/remediation/test-gen all call Anthropic; on an API error they degrade to
static defaults rather than failing over to another provider) — no custom model
training, no local inference. `OPENAI_API_KEY` exists in `apps/api` config as a
placeholder for a possible future fallback but is not read by any code path today;
adding real fallback would need the `openai` SDK as a new dependency, which
requires an ADR under the standing rule below before it's added.

**Module boundaries:** `apps/api/shield_api` (FastAPI backend), `apps/cli/shield`
(Typer CLI), `apps/web` (Next.js dashboard), `packages/scanner` (detector wrappers +
pipeline), `packages/normalizer` (finding normalization/dedup), `packages/ai-engine`
(triage/remediation/test-gen), `packages/github` (GitHub App + PR integration).

**Standing rule:** No agent may introduce a new external service, database, or
infrastructure dependency without logging the decision in a new
`docs/decisions/NNN-*.md` ADR and getting explicit human approval first.

---
> Source: [AliAmmar15/Velonus](https://github.com/AliAmmar15/Velonus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
