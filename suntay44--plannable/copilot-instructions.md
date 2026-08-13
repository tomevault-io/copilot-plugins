## plannable

> Plannable progressive planning rules


# Plannable Rules

- Humans read `MASTER_PLAN.md`.
- Always read `MASTER_PLAN.md` first.
- Agents read only the next pending `plans/PART*_PLAN.ai.md` file.
- Never load all part files unless the user explicitly asks.
- The compressed format is PlannablePlan, not PlanPack.
- Every `.ai.md` file starts with `@PlannablePlan v0.1`.
- A part is complete only after evidence is recorded in `PLAN_EVIDENCE.md`.
- Do not edit unrelated files.
- Stop on stop conditions in the active part.
- Update `PLAN_STATE.md` before final completion.
- Check off `MASTER_PLAN.md` only after evidence exists.
- Do not require `plannable init` before `plannable create`.
- Run `plannable verify` before reporting completion.

---
> Source: [suntay44/plannable](https://github.com/suntay44/plannable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
