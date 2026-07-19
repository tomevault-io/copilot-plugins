## specify-rules

> <!-- SPECKIT START -->


<!-- SPECKIT START -->
For additional context about technologies, project structure, shell commands, and
pipeline segments, read the current implementation plan:

**Plan**: `specs/023-capability-realignment/plan.md` (active) | Shipped: `specs/022-outcome-score-ladder/plan.md` | Base: `specs/001-sec-disclosure-rag/plan.md`

Quick reference:
- **Stack**: uv, custom-judge v2.0.1 draft, judge v3.1 VA-only task_success, **paper-v1.1** cohort-gated
- **Layers**: LLM `resolve_xbrl_facts` + Python compute; **no** live 022 heuristics; repro slice expansion retained
- **CLI**: `repro cohort-debug` → judge-batch → `specs/023-capability-realignment/scripts/audit_cohort_synthesis_paths.py`
- **Focus**: Principle VII realignment — single numeric path, filing catalog, taxonomy index, role-aware validation
- **Docs**: `specs/023-capability-realignment/quickstart.md`, baseline `reports/cohort-022-phase-e`, latest `reports/cohort-023-m4b`
<!-- SPECKIT END -->

---
> Source: [caldeirav/agentic-graphrag-finance](https://github.com/caldeirav/agentic-graphrag-finance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
