## crypto-anaylzer

> Run a portfolio-grade or hiring-manager-style audit on the Crypto-Analyzer repo: use a phased checklist across repo sanity, README, architecture, DB, dashboard, and tests; produce a punch list; apply fixes; and finish with a verification report and hiring-manager verdict. Use when the user asks for portfolio readiness, a final review, a ruthless audit, or a statsmodels-level quality pass.


# Portfolio / Hiring Manager Audit

## Hard Constraints
- Do not add new product features unless the user explicitly asks.
- Keep SQLite as the single source of truth.
- Do not add API keys or authenticated endpoints.
- Keep tests offline and deterministic.
- Preserve backward compatibility or clearly document migrations.

## Phased Checklist

### Phase 0 - Repo sanity
- Inspect the repo layout and entrypoints.
- Flag dead scripts, duplicate logic, unclear naming, or god files.

### Phase 1 - README quality
- Check that a non-technical reader can understand what the repo does and why.
- Check that an engineer can understand quickstart, architecture, provider model, data model, testing, and troubleshooting.
- Verify every README claim against code or tests.

### Phase 2 - Architecture verification
- Verify provider interfaces, registry wiring, config-driven chains, fallback behavior, retries, circuit breakers, last-known-good handling, and provenance.

### Phase 3 - DB and migrations
- Confirm migrations are idempotent and non-destructive.
- Confirm schema and write paths preserve provenance, health, and auditability.

### Phase 4 - Dashboard and models
- Confirm the dashboard shows freshness and provider health clearly.
- Confirm the modeling path stays deterministic and does not silently mix sources.

### Phase 5 - Tests and tooling
- Run deterministic offline tests.
- Check provider fallback, retry, circuit-breaker, and provenance coverage.
- Run lint and document any gaps.

### Phase 6 - Hiring-manager pass
- Judge whether the repo feels clean, trustworthy, documented, and easy to extend.

## Final Output
1. Punch list with severity, issue, and location
2. Fixes applied
3. Verification report with exact commands and results
4. Hiring-manager verdict with remaining risks
5. Extensibility proof if requested

## Verification Commands
- `python -m pytest -q`
- `ruff check .`
- `.\scripts\run.ps1 poll` or equivalent poll smoke command when ingestion changed
- `.\scripts\run.ps1 streamlit` when dashboard behavior matters

---
> Source: [AlpharomeroJL/Crypto-Anaylzer](https://github.com/AlpharomeroJL/Crypto-Anaylzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
