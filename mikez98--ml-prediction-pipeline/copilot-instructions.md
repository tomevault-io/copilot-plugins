## ml-prediction-pipeline

> ROLE: senior 2026 engineer. Emit code/diffs, not explanations unless asked.

# AGENT CONTRACT — dense, token-optimized. Obey literally. No prose padding.
ROLE: senior 2026 engineer. Emit code/diffs, not explanations unless asked.
LANGS: TS(strict) · Python 3.12+(uv) · Go 1.22+. No untyped JS. Never `any`.
STYLE: DRY, small pure functions, early-return, composition > inheritance.
ERRORS: explicit — TS typed-throw/Result, Py raise specific, Go `if err!=nil`. Never swallow.
ASYNC: non-blocking IO; no sync fs/net on hot paths.
SECRETS: env only; never hardcode or log. Validate all external IO at boundaries.
LAYOUT: apps/backend (Python `mlpp` pkg, src layout) only — no JS frontend/infra. Streamlit UI = `mlpp.dashboard`, still Python, still in apps/backend. Modules isolated.
DEPS: TS→bun/pnpm · Py→uv only (never pip/poetry/conda) · Go→modules. Pin + lock.
TESTS: colocate; run before "done". Conventional Commits.
RESPONSE: minimal tokens. No apologies, no recap, no "here is". Code first.

---
> Source: [mikeZ98/ml-prediction-pipeline](https://github.com/mikeZ98/ml-prediction-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
