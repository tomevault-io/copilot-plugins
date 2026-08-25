## spine

> This is project aiming to develop a spec drivend agentic workflow

This is project aiming to develop a spec drivend agentic workflow

- when developing a feature that needs harness specific stuff - create a compatibility layer for codex / calude code/ opencode (or 3 separate implementations) to cover compatibility with those

---

# <!-- BEGIN PROJECT HOMONTO -->

## Project Homonto

Homonto is opt-in. Use it only when the user invokes `$homonto-spec`, `$homonto-pwf`, or
explicitly asks to use Homonto. Load shared semantics from `.agents/skills/`; otherwise work
normally.

Feature-local `state.json` is the only workflow-state authority. Explicit spec and plan
approval must be recorded there with approver and approved revision before advancing.
Autonomy never waives approval or independent-verification gates unless explicitly auto mode is required.

`spec.md` owns required behavior and product decisions. `plan.md` references spec IDs and
owns technical approach, change sequence, risk, and proof mapping. Do not duplicate the spec
in the plan or require a fixed prose shape.

Verification must run in a fresh harness context, inspect the complete diff, execute checks
directly, and write revision-bound `verification.json` evidence. If the active harness cannot
provide the required isolation, stop rather than self-verify. Verification and closeout are
single-pass assessment steps: do not edit implementation, delegate repairs, invent requirements,
or autonomously enter fix/reverify loops. Report evidenced blockers and stop; remediation requires
a separate, explicitly authorized implementation attempt. Treat speculative hardening as
non-blocking follow-up work.

# <!-- END PROJECT HOMONTO -->

---
> Source: [prostakm/spine](https://github.com/prostakm/spine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
