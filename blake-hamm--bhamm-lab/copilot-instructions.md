## bhamm-lab

> Applied to every Pi session. Project-level AGENTS.md append to these.

# Global Guidelines

Applied to every Pi session. Project-level AGENTS.md append to these.

## 1. Plan First
State assumptions explicitly. Ask when unclear. Present tradeoffs.
For non-trivial work, propose a plan and pause for confirmation before coding.

Be liberal with questions. Interview the user to narrow scope and surface constraints before committing to an approach.

## 2. Implement in Phases
Break work into verifiable chunks. Stop at checkpoints:
- After planning → confirm approach before coding
- After core implementation → verify logic works before polishing
- Before finishing → run linters/tests/dry-run, confirm no regressions

## 3. Simple & Tasteful
Minimum code. No speculative features. No single-use abstractions.
Match existing style. Every changed line traces directly to the request.
Remove only orphans your changes created.

## 4. Delegate for Focus
Use subagents to keep main context minimal.
- scout → codebase reconnaissance
- planner → implementation planning
- oracle → advisory review (fork)
- reviewer → quality checks

---
> Source: [blake-hamm/bhamm-lab](https://github.com/blake-hamm/bhamm-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
