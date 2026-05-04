## codex-1up

> Early development, no users. No backwards compatibility concerns. Do things RIGHT: clean, organized, zero tech debt. Never create compatibility shims.

# Repository Guidelines

Early development, no users. No backwards compatibility concerns. Do things RIGHT: clean, organized, zero tech debt. Never create compatibility shims.

This repository provides a lightweight installer and helpers that “power up” the OpenAI Codex CLI. It is primarily TypeScript (CLI + zx runtime) with a few Bash helper scripts.

## Project Structure & Module Organization
- `bin/codex-1up` — user‑facing wrapper for common tasks (`install`, `agents`, `doctor`, `uninstall`).
- `cli/src/installers/` — Node/zx installer modules invoked by the `install` command.
- `scripts/doctor.sh` / `scripts/uninstall.sh` — health checks and cleanup.
- `templates/` — Codex config and `AGENTS.md` templates; subfolders for languages.
- `public/` — assets (e.g., `banner.png`).
- `docs/` — additional documentation.

## File Extensions, Folders & Globs
- Shell scripts: `*.sh`
- Markdown: `*.md`
- TOML configs: `*.toml`
- Fish shell: `*.fish`
- Images: `*.png`
- Folders: `bin/`, `scripts/`, `templates/`, `templates/agent-templates/`, `templates/configs/`, `templates/shell/`, `public/`, `docs/`

## Build, Test, and Development Commands
- `./bin/codex-1up install --dry-run` — show planned actions without changing the system.
- `./bin/codex-1up install --yes` — perform a non‑interactive install with safe defaults.
- `./bin/codex-1up doctor` — verify prerequisites and report environment findings.
- `./bin/codex-1up agents --path $PWD --template default` — write a starter `AGENTS.md` to a target repo.
- `./bin/codex-1up uninstall` — remove aliases/config this tool created (does not uninstall packages).

## Coding Style & Naming Conventions
- Languages: TypeScript (Node ESM) and Bash helpers.
- TS: 2‑space indent; keep imports ESM; prefer small modules; run via `tsx` in dev and build with `tsup`.
- Bash: `#!/usr/bin/env bash` with `set -euo pipefail`; 2‑space indent; no hard tabs.
- Files and functions: kebab‑case for scripts, `lower_snake_case` for shell functions/vars.
- Prefer POSIX utilities; when faster tools exist, use: `fd` for file search, `rg` for text, `jq`/`yq` for JSON/YAML.
- Validate shell with `shellcheck` and format with `shfmt -i 2 -ci`.

## Testing Guidelines
- No formal test suite; validate changes via:
  - `./bin/codex-1up install --dry-run` and `./bin/codex-1up doctor` on macOS + Linux.
  - Simulate branches with env flags; avoid mutating global state in tests.
- Add minimal, reproducible examples to PRs (command + observed/expected output).

## Commit & Pull Request Guidelines
- Use Conventional Commits when practical: `feat:`, `fix(scope):`, `docs:`, `refactor:`.
- Commits should be small and focused; avoid mixing functional changes with formatting.
- PRs must include:
  - Summary of changes and rationale.
  - Platform tested (e.g., macOS 14, Ubuntu 22.04) and `doctor` output excerpt.
  - Screenshots or terminal snippets when UX changes.
  - Notes on risk/rollback (backups are created for overwritten files).

## Security & Configuration Tips
- Prefer `--dry-run` first; review changes before applying.
- Avoid running with elevated privileges; the scripts do not require `sudo`.
- Backups are created on overwrite; verify paths before confirming prompts.

---
> Source: [regenrek/codex-1up](https://github.com/regenrek/codex-1up) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
