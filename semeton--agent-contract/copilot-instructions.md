## agent-contract

> <!-- agent-contract:shim -->

<!-- agent-contract:shim -->
# Agent Contract — ALWAYS ACTIVE

This repository enforces a deterministic agent contract. The contract is **non-optional** and applies to every session, every task, every model. It does not need to be mentioned in the prompt — it is always in effect.

## Boot Sequence — Run at the Start of EVERY Session

1. Check `.agent/memory/` for any `handoff-*.md` files. If found, read the most recent before proceeding.
2. Read `.agent/manifest.yaml` to load the full contract.
3. Read `.agent/memory/codebase-map.md` — use this as your index. Do NOT read raw source files before consulting the map.
4. Load the relevant role from `.agent/roles/<role>.yaml` for the current task.
5. Run `.agent/checks/pre-generate.sh` before generating any code.
6. Write your active role name (e.g. `generator`) to `.agent/session/active-role.txt` — one word, no whitespace. This enables the scope-enforcement hook.

## Enforcement Rules

- Scope violations **hard-fail**. Only touch files within the declared task scope.
- When inputs are missing or ambiguous, **ask the human** — do not assume or fill in defaults.
- `refactor_policy: forbidden_unless_explicitly_approved` — fix only what was asked.
- Log every non-trivial decision to `.agent/memory/decisions.jsonl`.

## Session Handoff Protocol

When a task is **completely done** (checks pass, decisions logged):

1. Write a handoff note to `.agent/memory/handoff-YYYY-MM-DD-HHMM.md` using `.agent/templates/handoff.md` as the template.
2. Tell the user: "Task complete. Start a fresh session and open `.agent/memory/handoff-<timestamp>.md` to continue."

Handoff notes are the only continuity guarantee across sessions. Always write one at task completion.
<!-- /agent-contract:shim -->

---
> Source: [Semeton/agent-contract](https://github.com/Semeton/agent-contract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
