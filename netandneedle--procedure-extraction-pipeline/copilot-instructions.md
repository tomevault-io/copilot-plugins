## procedure-extraction-pipeline

> Automated ATT&CK procedure extraction: threat reports in, validated STIX 2.1

# CLAUDE.md

Automated ATT&CK procedure extraction: threat reports in, validated STIX 2.1
bundles with `x-procedure` objects out, with four human review gates. Python
(FastAPI + LangGraph) backend, React frontend, Postgres + Neo4j.

@docs/ARCHITECTURE.md
@CONTRIBUTING.md

Two test suites, two configs: `pytest` at the repo root runs `tests/` only;
`cd backend && pytest` runs `backend/tests/`. Both must pass. There is no
lint or typecheck step.

---
> Source: [netandneedle/procedure-extraction-pipeline](https://github.com/netandneedle/procedure-extraction-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
