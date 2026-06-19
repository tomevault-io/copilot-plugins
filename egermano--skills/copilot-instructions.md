## skills

> - This repo is a lightweight personal archive of copied skill docs; there is no application code, build system, test suite, or package manager config.

# AGENTS

## Repository Reality
- This repo is a lightweight personal archive of copied skill docs; there is no application code, build system, test suite, or package manager config.
- Current tracked content is `README.md` plus skill copies under `skills/*/SKILL.md`.

## Working Rules For Agents
- Treat files in `skills/<skill-name>/SKILL.md` as source copies; prefer minimal edits and avoid rewriting style/structure unless explicitly requested.
- Use `README.md` and this file as the only local instruction sources; there are currently no `.github/workflows/`, `opencode.json`, `CLAUDE.md`, or other repo-local agent rule files.
- Do not invent validation steps (lint/test/build) for this repo; verify changes with file-level checks and `git status`.

## Common Commands
- Check repo state: `git status -sb`
- Inspect tracked files quickly: `git ls-files`
- Add a new copied skill folder: `mkdir -p skills/<skill-name>` then copy `SKILL.md` into it

## Commit / PR Notes
- Branch is `main` and tracks `origin/main`.
- Keep commits scoped to documentation/content operations (add/update copied skills, update indexes like `README.md`).

---
> Source: [egermano/skills](https://github.com/egermano/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
