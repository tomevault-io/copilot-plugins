## dotfiles

> This is a personal dotfiles repository. Keep workflows simple and practical.

# Project Instructions for AI Agents

This is a personal dotfiles repository. Keep workflows simple and practical.

## Task Tracking

- Formal issue tracking is optional.
- Small config tweaks, docs edits, and one-off script fixes do not need `bd`.
- Use `bd` only for larger multi-step work, blocked work, or backlog items worth revisiting.
- If `bd` is unavailable, continue without it.

Optional `bd` examples:

```bash
bd ready --json
bd create "Issue title" --description="Context" --json
bd update <id> --claim --json
bd close <id> --reason "Completed" --json
```

## Build & Test

Use lightweight validation appropriate to the change.

```bash
bash -n scripts/*
just --list
./scripts/bootstrap --dry-run
```

Run only the commands relevant to the files you changed.

## Architecture Overview

- `config/` contains files symlinked into `$HOME`.
- `scripts/` contains repo tooling for bootstrap, linking, and adoption (absolute symlinks; no Stow or chezmoi).
- `Justfile` is the main command surface for common tasks.
- `makefile` is a thin wrapper over the same scripts.
- `docs/` contains setup notes and personal reference docs.

## Conventions & Patterns

- Prefer `just` recipes for documented workflows.
- Prefer small shell scripts over heavy tooling.
- Keep docs aligned with actual commands and file mappings.
- For symlink-managed files, use `just sync` after pulling new repo changes.
- Avoid introducing process overhead unless it clearly helps.

## Session Completion

When ending a work session:

1. Run relevant validation for the changes made.
2. Summarize any remaining follow-up work in the handoff.
3. Push all committed changes:

```bash
git pull --rebase
git push
git status  # must show up to date with origin
```

If `bd` was used and is configured, optionally run:

```bash
bd dolt push
```

## Critical Rules

- Work is not complete until `git push` succeeds.
- Do not leave committed work only in the local repo.
- Do not make optional tooling a blocker for routine dotfile maintenance.

---
> Source: [pazthor/dotfiles](https://github.com/pazthor/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
