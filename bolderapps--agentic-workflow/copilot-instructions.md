## 01-verification

> Verification gates that must pass before any task is handed to QA.


# Verification gates

## When to run

Builders run these gates before marking any task `in-review`.
QA runs these gates as part of its review.
Never hand off without all gates passing.

## Gate 1 — Lint

Run the project linter on all files created or modified in the current task.

- Zero errors allowed.
- Zero warnings allowed unless the warning is pre-existing and unrelated to this task.
- If the project has no linter configured yet: note it in `memory/decisions.md` and skip.

## Gate 2 — Typecheck

Run the TypeScript compiler (or equivalent) on the full project.

- Zero type errors allowed.
- `tsc --noEmit` or equivalent.
- Fixing a type error by adding `as any` or `@ts-ignore` is not acceptable unless the reason is documented in a comment.

## Gate 3 — Tests

Run tests scoped to the feature being built.

- All existing tests must continue to pass (no regressions).
- All new tests introduced by the task must pass.
- Minimum test coverage for the task: happy path + at least one error/edge case per unit.
- If a test runner is not yet set up: the scaffold task must include setting it up before any feature task starts.

## Gate 4 — Acceptance criteria

Every checklist item in the task block must be verifiable.

- Each `[ ]` item in the acceptance criteria must be checked `[x]`.
- If an item cannot be verified automatically, the QA agent documents how it was verified manually.

## Non-negotiable rule

**If any gate fails, the task is not done.** Fix in the current session before handing off. Do not move to the next task while a gate is failing.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
