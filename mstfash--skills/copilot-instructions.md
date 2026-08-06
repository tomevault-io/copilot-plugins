## skills

> Operating instructions for AI agents working in this repo.

# AGENTS.md

Operating instructions for AI agents working in this repo.

## Before marking any unit of work "done"

**Run the checklist in [PHASE_CHECKLIST.md](./PHASE_CHECKLIST.md) to green first.**
It is the definition of done, not optional polish. Read it at the moment of use —
don't work from memory — and follow every step in order, including the loops.

Why: passing tests and a self-review is not proof the work is correct. The author
shares their own blind spots, so each step in the checklist exists to catch a class
of problem the cheaper steps miss. Do not self-certify; run the gate.


# global agent instructions

- Never use the em dash "—". Use plain dash "-" instead
- When writing commit messages, NEVER auto-add your agent name as co-author
- Never manually modify CHANGELOG.md files or any files that are marked as auto-generated
- When making technical decisions, do not give much weight to development cost.
- Instead, prefer quality, simplicity, robustness, scalability, and long term maintainability.
- When doing bug fixes, always start with reproducing the bug in an E2E setting as closely aligned with how an end user would experience it as possible.
  This makes sure you find the real problem so your fix will actually solve it.
- When end-to-end testing a product, be picky about the UI you see and be obsessed with pixel perfection.
- If something clearly looks off, even if it is not directly related to what you are doing, try to get it fixed along the way.
- Apply that same high standard to engineering excellence: lint, test failures, and test flakiness.
- If you see one, even if it is not caused by what you are working on right now, still get it fixed.

---
> Source: [mstfash/skills](https://github.com/mstfash/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
