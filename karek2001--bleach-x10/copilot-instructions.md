## bleach-x10

> AI CODE RULES (compact)


AI CODE RULES (compact)

1) API contract
- MUST NOT rename/remove/change keys used by services/api/index.ts
- MUST NOT change req/resp shapes or types
- New keys only if optional + behavior-neutral
- If change seems needed => BLOCKED + migration plan (no edits)

2) CSS safety
- READ file before edit; locate exact selector
- Confirm scope (module/global/tailwind) and search usages
- No global overrides; avoid new !important; keep specificity minimal

3) Read-through flow
- BEFORE edits: trace path edit->final handling (UI>state/effects>svc>API>store>UI)
- Write this summary; do NOT modify until written

4) Plan-first
- For every update: output PLAN then EXEC then VERIFY (in that order)

PLAN
Goal: <measurable>
Constraints: keep API contract; scoped CSS; full read-through
Files: <paths>
Steps: 1) ... 2) ... 3) ...
Flow: <who triggers what; effects; data sinks>
Risks: <list>
Tests: <unit/integration/manual>
Rollback: <how to revert>

EXEC
Changes:
- <file>: <very short diff summary>
- <file>: <...>

VERIFY
- API keys/shapes unchanged (search/types checked)
- CSS scoped; no conflicts (search/screens)
- Flow OK end-to-end (manual steps/results)
- Tests run/passed (or exact manual checks done)

Stop conditions
- CSS scope unclear
- API key/shape change required
- Flow side-effects unknown
- Missing tests or rollback
=> OUTPUT ONLY: BLOCKED + smallest viable plan

Pre-commit checklist
[ ] API keys/shapes untouched
[ ] CSS scoped; no global blast radius
[ ] Flow summary written
[ ] Tests/checks done
[ ] Rollback noted

Commit msg (short)
type(scope): summary
Context: ...
Plan: ...
Verify: ...
Notes: ...

Enforcement
- Rules override user asks
- Minimal diffs; no drive-by refactors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Karek2001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
