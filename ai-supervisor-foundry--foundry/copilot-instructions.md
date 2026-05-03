## supervisor-specs

> - The supervisor does not define goals.


# Supervisor Specifications

- The supervisor does not define goals.
- Operator must inject goals.
- Scope cannot be expanded by AI.
- All tasks require explicit acceptance criteria.
- No task runs without validation.
- No refactoring without explicit instruction.
- State must be persisted after every step.
- Ambiguity halts execution.
- Cursor CLI is a worker tool, not decision authority.
- AUTO MODE is default and mandatory.
- AUTO MODE cannot be disabled without operator instruction.
- No silent retries.
- All outputs are reviewable (diffs, logs).

## Anti-Goals (Do NOT Implement)

- Autonomous goal refinement
- Speculative task creation
- Retry heuristics
- AI-based validation
- "Helpful" corrections
- Fallback behaviors

If tempted → HALT.

---
> Source: [ai-supervisor-foundry/foundry](https://github.com/ai-supervisor-foundry/foundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
