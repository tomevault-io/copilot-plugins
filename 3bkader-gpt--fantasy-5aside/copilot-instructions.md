## fantasy-5aside

> Use sequential thinking for non-trivial tasks (plan, then execute with verification gates).


# Sequential thinking workflow (soft guidance)

When a task has 3+ steps or touches multiple modules:

- Start with a **short plan**:
  - goal, constraints, key files, verification steps.
- State any assumptions explicitly and validate them quickly.
- Execute in **small diffs**; avoid “mega changes”.
- Add **verification gates**:
  - run `pytest` (or targeted tests) after code changes,
  - validate key user flows when changes affect UX.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/3bkader-gpt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
