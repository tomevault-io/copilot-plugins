## pr-map

> These rules govern any agent implementing tasks from `plan.md` (e.g. under the `ralph` loop).

# Autonomous implementation contract

These rules govern any agent implementing tasks from `plan.md` (e.g. under the `ralph` loop).
They are mandatory, and `/review` enforces them — a violation is a defect, not a style note.

## Start of each iteration — read the context

Before picking a task, read `plan.md` and `learnings.txt` in full. **Also read `review-notes.txt` if it
exists** and treat its `[REVIEWER …]` entries as course-corrections with the same weight as
`learnings.txt` — fix what they call out before adding new work. (`review-notes.txt` is written by the
`/deliver` parallel reviewer; it may not exist outside a `/deliver` run — read it only if present.)

## Test-Driven Development is non-negotiable

Every task follows RED -> GREEN -> REFACTOR:

1. **RED** — write a failing test first that encodes the task's acceptance criterion. Run it and
   confirm it fails for the intended reason. No production code before a failing test exists.
2. **GREEN** — write the minimal code to make the test pass. No placeholders, stubs, TODOs, or
   commented-out code.
3. **REFACTOR** — improve structure without changing behavior; keep tests green.

## Definition of Done (before every commit)

A task is done only when ALL three gates pass — run the exact commands from plan.md's
`## Verification commands`:

- **Tests** — the relevant suite is green, including the new test.
- **Lint** — clean.
- **Type check** — clean.

If any gate is red, fix it before committing. Never commit a red gate.

## Test integrity

- Tests must assert real behavior tied to the acceptance criterion.
- Never make a test pass by weakening it, deleting assertions, marking it `skip`/`xfail`/`only`,
  or loosening the gate config.
- A green suite achieved by gaming it is a defect, not "done".
- Test observable behavior through the public interface, not implementation details. A test that
  breaks on a behavior-preserving refactor was testing the wrong thing.
- No tautological tests: the expected value must come from an independent source — a known-good
  literal or a worked example from the spec — never recomputed the way the code computes it.
- Mock only at true system boundaries (external APIs, DB, time, randomness, filesystem). Never
  mock internal collaborators or your own modules.

## Scope discipline

- One task = one commit. Do not batch tasks.
- Obey plan.md's `## Constraints`. Do not refactor adjacent code, add unrequested features, or gold-plate.
- Follow the patterns plan.md references — read the referenced files and replicate their structure.
- Leave the codebase in a working state after every commit.

## Testability & slicing

- A task must be testable in isolation through its **public interface** with the code that exists after
  it. Prefer **dependency ordering** — build a collaborator before the code that needs it — so the test
  exercises real behavior, not a mock. This is the same rule as "never mock internal collaborators."
- Integration behavior belongs in its own dependency-ordered task with a real integration test that
  spans the now-built collaborators — not smuggled into an earlier task via fakes.
- If a task cannot be tested in isolation without faking an internal collaborator, **STOP and record it
  in learnings.txt for re-slice** (reorder so the collaborator comes first, or split out an integration
  task). Do not mock the collaborator and do not fabricate a test to move on.

## When stuck

If a task is ambiguous or needs a design decision plan.md does not answer, stop and record the
question in learnings.txt rather than guessing.

---
> Source: [ValentinDutra/pr-map](https://github.com/ValentinDutra/pr-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
