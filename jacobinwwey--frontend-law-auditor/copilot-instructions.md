## frontend-law-auditor

> You are working with the `frontend-law-auditor` skill package.

# Cursor Rules for Frontend Law Auditor

You are working with the `frontend-law-auditor` skill package.

## Global Rules
1. Before auditing a UI, read the repository-root `SKILL.md`.
2. Use `python3 scripts/law_audit.py` for template generation, scoring, strict gating, and report output.
3. Load supporting documents only when needed:
   - `references/metrics-schema.md`
   - `references/theory-playbook.md`
   - `rules/_sections.md`
   - `rules/<law>.md`
   - `examples/evidence.sample.json`
4. Never invent measurements or observations. Missing evidence must remain `unknown`.
5. The final audit output must include:
   - Fast Gate Failures
   - Law Diagnosis
   - Priority Fix Plan
   - Recheck Checklist

## Activation Map
- Audit a UI, onboarding flow, checkout, settings page, or form usability: read `SKILL.md` first.
- Build a measurable frontend release gate or CI policy: read `SKILL.md` first.
- Explain why a design violates Fitts, Hick, Gestalt, Jakob, Miller, or related laws: read `SKILL.md` first.

---
> Source: [Jacobinwwey/frontend-law-auditor](https://github.com/Jacobinwwey/frontend-law-auditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
