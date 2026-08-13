## aloratech

> This repository is a set of Claude rules, skills, and agents for working on the project. The rules below are **binding** — read and follow them.

# Project Instructions

This repository is a set of Claude rules, skills, and agents for working on the project. The rules below are **binding** — read and follow them.

## Rules

@.claude/rules/workflow.md

@.claude/rules/security-hygiene.md

@.claude/rules/stack.md

## How the pieces fit together

- **Rules** (`.claude/rules/`) — the binding policy. `workflow.md` governs how work flows (orchestration discipline → TDD → Claude review → finish). `security-hygiene.md` governs what may be committed under `.claude/`. `stack.md` pins the assumed tech stack and its conventions — when reusing this config on a different stack, edit only that file.
- **Skills** (`.claude/skills/`) — `design`, `commit`, `claude-review`, `codex-review`, `create-pr`. Invoke via the Skill tool / slash commands. Run `design` **before** building any new screen/component (ASCII-wireframe-first).
- **Agents** (`.claude/agents/`) — review panel: `dry-checker`, `security-reviewer`, `api-checker`, `general-code-reviewer`, `design-reviewer`. Dispatched in diff-scoped mode by `create-pr`, or manually for full-project sweeps.

---
> Source: [cbpcanda/aloratech](https://github.com/cbpcanda/aloratech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
