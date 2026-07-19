## agent-capability-first

> Capability-first agent design — prefer prompts/skills over keyword handlers


# Agent Capability-First Design (Principle VII)

When improving agentic retrieval—especially numeric XBRL synthesis—follow this ladder:

## Decision ladder

1. **Structured output** — JSON answer contract → rendered prose; ban `"Based on N evidence chunk(s)"` in live paths (`src/retrieval/skills/structured_answer.py`).
2. **Prompt enrichment** — Pass benchmark `temporal_anchor` and `fiscal_period_labels` into macro planner and synthesis (`runner.py` → `service.py` → `macro_router` / `synthesis.py`).
3. **LLM skills** — Disambiguation before synthesis (e.g. `src/retrieval/skills/xbrl_fact_resolution.py`).
4. **Cohort gate** — Validate on frozen cohort (`xbrl_numeric_cohort.json`) via `repro cohort-debug` before full repro.

## Do not (without ADR)

- Add new `_try_synthesize_*` keyword routers as the primary fix for financebench failures.
- Fall back to `_synthesize_template` chunk dumps when `USE_MOCK_LLM` is unset.

## Allowed

- Deterministic handlers when `USE_MOCK_LLM=1` (CI/fixtures).
- Documented exceptions in plan Complexity Tracking.

## Reference

- Spec: `specs/020-agent-capability-first/spec.md`
- Quickstart: `specs/020-agent-capability-first/quickstart.md`
- Constitution: `.specify/memory/constitution.md` Principle VII

---
> Source: [caldeirav/agentic-graphrag-finance](https://github.com/caldeirav/agentic-graphrag-finance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
