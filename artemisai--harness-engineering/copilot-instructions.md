## harness-engineering

> For full context, read `AGENTS.md` first. This file adds Copilot-specific configuration.

# GitHub Copilot Instructions — Harness Engineering

For full context, read `AGENTS.md` first. This file adds Copilot-specific configuration.

## Session Protocol

Every session:
1. Read `claude-progress.txt` (last 30 lines)
2. Read `features.json` — target the highest-priority `"failing"` feature
3. Confirm one feature is in scope before writing code
4. End with a `claude-progress.txt` entry and updated `features.json`

## Code Style

- Commits: `feat(<id>): <description>` / `fix(<id>): ...` / `chore(entropy): ...`
- Branch naming: `feat/<id>-<short-description>`
- No WIP commits on `main`

## Skills

Reference skills with `@skill-name` in Copilot chat. Skills live in `skills/*.skill.md`.

Available:
- `@session-start` — orient to current project state
- `@session-end` — clean handoff before closing
- `@implement-feature` — spec-driven implementation
- `@new-feature` — scaffold a new feature
- `@run-tests` — focused test run
- `@create-issue` — log bug or blocker
- `@delegate-subagent` — spawn a subagent in a worktree
- `@entropy-check` — find and fix accumulated drift
- `@write-adr` — record an architecture decision

## Constraints

Never run: `rm`, `del`, `rmdir`, `kill`, `sudo`, `shutdown`
Never use: `&&`, `||`, `..` in shell commands
One feature per session — no scope creep.

---
> Source: [ArtemisAI/Harness_Engineering](https://github.com/ArtemisAI/Harness_Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
