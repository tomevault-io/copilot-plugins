## harness-for-codex

> This repository is a baseline workspace for Codex-driven implementation.

# Codex Harness Instructions

This repository is a baseline workspace for Codex-driven implementation.

## Commands

Run these from the repository root:

```sh
scripts/bootstrap
scripts/check
scripts/test
scripts/eval
scripts/doctor
scripts/hooks
```

- `scripts/bootstrap`: prepare dependencies when a known stack is present.
- `scripts/check`: run lint, type checks, formatting checks, and tests when available.
- `scripts/test`: run only the test suite when available.
- `scripts/eval`: run the complete handoff verification sequence.
- `scripts/doctor`: report repository, tool, and environment readiness.
- `scripts/hooks`: install local Git hooks through `pre-commit` when available.

## Repository Layout

- `AGENTS.md`: Codex instructions for this repo.
- `CLAUDE.md`: Claude Code bridge that imports `AGENTS.md`.
- `README.md`: Human-facing overview and workflow.
- `harness.yml`: Machine-readable harness metadata and command registry.
- `.devcontainer/`: Optional reproducible development container.
- `.pre-commit-config.yaml`: Optional local hook definitions.
- `docs/`: Project notes and decisions.
- `scripts/`: Stable automation entrypoints.
- `tasks/`: Task briefs and working notes.

## Operating Principles

- Read the repository before editing. Prefer existing conventions over new ones.
- Keep changes scoped to the requested task.
- Do not revert user changes unless explicitly asked.
- Use `rg` for searching when available.
- Use the standard scripts instead of one-off local commands when they cover the task.
- Add or update tests when behavior changes.
- Document decisions that affect future work in `docs/decisions.md`.
- Add nested `AGENTS.md` files in subdirectories only when that area needs different instructions.
- Keep root-level agent instructions short and durable. Move long procedures into docs or scripts.

## Task Loop

1. Inspect: read relevant files and recent decisions.
2. Plan: identify the narrow change and verification path.
3. Implement: make focused edits.
4. Verify: run `scripts/check` or `scripts/eval`.
5. Handoff: summarize changed files, verification, and residual risks.

## Cross-Agent Compatibility

- Codex and Cursor can read `AGENTS.md`.
- Claude Code reads `CLAUDE.md`, which imports `AGENTS.md`.
- Keep shared instructions in `AGENTS.md` unless a tool-specific note is required.

## Script Policy

These scripts are intentionally stack-aware and conservative. If a language stack
is added later, extend the scripts rather than bypassing them.

## Completion Criteria

Before finishing a task:

1. Confirm the requested change is implemented.
2. Run the narrowest useful verification command, usually `scripts/check`.
3. Report changed files and any verification that could not be run.

---
> Source: [ganimjeong/Harness-for-codex](https://github.com/ganimjeong/Harness-for-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
