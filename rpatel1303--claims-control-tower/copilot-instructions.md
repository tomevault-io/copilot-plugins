## claims-control-tower

> This file defines standards for both Codex and Claude Code in this repo.

# Agent Working Agreement

This file defines standards for both Codex and Claude Code in this repo.

## Scope
- Applies to all AI agents making changes in this repository.
- If conflicts exist, this file overrides other guidance.

## Branch Naming
- Use `codex-*` for Codex work.
- Use `claude-*` for Claude Code work.

## Commit Format
- Use Conventional Commits.
- Example: `feat: add executive scorecard`

## Story Workflow
- Always create a story before making any changes.
- All work must start from a story in `claims-control-tower/PROJECT.md`.
- Use the story template in `docs/STORY_TEMPLATE.md`.
- Create a dedicated branch per story.
- Update story status when work starts and when it completes.
- After work is done and signed off, commit changes and delete the story branch.

## Shared Commands
- Follow `docs/COMMANDS.md` for cross-agent slash commands.

## Build and Tests
- Run tests/build checks for every commit.
- If no tests exist, state "No tests available" in the final update.

## Documentation
- Document new features and changes in `claims-control-tower/PROJECT.md`.
- If behavior changes, update relevant docs in the repo.

## Never Do (Standard)
- Never use destructive git commands (`git reset --hard`, `git checkout --`) unless explicitly approved.
- Never revert unrelated changes.
- Never amend commits unless explicitly requested.
- Never assume network access; request approval when needed.

---
> Source: [rpatel1303/claims-control-tower](https://github.com/rpatel1303/claims-control-tower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
