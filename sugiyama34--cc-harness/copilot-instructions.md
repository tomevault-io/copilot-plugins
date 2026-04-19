## cc-harness

> <!-- Rewrite this section for your project. Example: -->

# CLAUDE.md

## Project Overview

<!-- Rewrite this section for your project. Example: -->
<!-- A web API for managing team schedules, built with FastAPI and PostgreSQL. -->

## Collaboration Style

The developer on this project is a beginner. Do not take instructions at face value — act as a knowledgeable collaborator:

- If a request deviates from best practices, say so and propose a better alternative.
- If the best approach is unclear, suggest researching before acting.

## Tech Stack

<!-- List your project's tech stack here. Example: -->
<!-- - **Language**: Python 3.12+ -->
<!-- - **Framework**: FastAPI -->
<!-- - **Database**: PostgreSQL -->
<!-- - **Package Manager**: uv -->

## Project Structure

- `.claude/hooks/` — Claude Code PreToolUse hook scripts.
- `.claude/skills/` — Claude Code skill definitions.
- `.claude/settings.json` — Permissions, sandbox rules, and hook registration.
- `.githooks/` — Git hooks (pre-commit secret scanner).
- `.github/workflows/` — CI/CD workflows.
- `scripts/` — Setup and utility scripts.
- `tests/hooks/` — Bats-core tests for all hook scripts.
- `docs/adr/` — Architecture Decision Records.
- `docs/claude/` — Extended documentation referenced from CLAUDE.md.
- `docs/logs/` — Daily work logs (`YYYY-MM-DD - Summary.md`).
- `docs/plans/` — Feature planning docs.

## Development Commands

```bash
bats tests/hooks/          # Run hook tests
./scripts/install-hooks.sh # Set up git hooks
```

## Git Workflow

- Never commit directly to `main`. Create a feature branch for each change.
- Branch naming: `feature/<short-description>`, `fix/<short-description>`, `docs/<short-description>`, etc.
- All changes go through: branch → commit → push → PR → review → merge.

## Localization

Some files have localized counterparts (e.g., `README.md` ↔ `README.ja.md`). When modifying a file with a localized version, always update both in the same commit.

## Error Messages

All error messages (hooks, scripts) must be actionable:

1. **Why** — what went wrong
2. **Claude Code** — what to do next
3. **User** — how to resolve it

## Shell Quoting for `gh api`

The command allowlist validates `gh api` calls using single-quoted patterns. Always use single quotes for `gh api` arguments (especially `-f body='...'`). To include an apostrophe inside a single-quoted string, use the `'\''` escape sequence.

## CLAUDE.md Management

- Keep this file under 200 lines.
- When it exceeds 200 lines, extract sections into `docs/claude/` and add a reference here.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sugiyama34) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
