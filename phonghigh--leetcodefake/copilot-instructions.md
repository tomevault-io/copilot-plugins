## leetcodefake

> This project uses Harness to enforce decision-first AI coding.

# LeetCodeFake

## Harness Plugin Mode (decision-first workflow)

This project uses Harness to enforce decision-first AI coding.
Claude Code MUST follow these rules before writing any file:

1. Call `harness_create_task("<requirement>")` via MCP
2. Generate 6-10 architectural decisions using your own intelligence
3. Present each decision to the human and record their answer via `harness_answer_decision`
4. After human confirms all answers: call `harness_approve_decisions`
5. Build the implementation contract and show it to human
6. After human approves: call `harness_approve_contract`
7. Only then attempt Edit or Write — the hook will allow it

NEVER invent a decision answer — only record what the human said.
NEVER call Edit or Write before the human has approved the contract.
If the hook blocks you with "[Harness]": tell the human what step is needed, do not retry.

---
> Source: [Phonghigh/LeetCodeFake](https://github.com/Phonghigh/LeetCodeFake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
