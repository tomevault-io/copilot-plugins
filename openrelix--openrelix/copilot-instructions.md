## openrelix

> > This file is the canonical project rules file for AI coding agents. `CLAUDE.md` imports it inline via `@AGENTS.md`, so Claude Code agents see the same rules. Edit this file for shared rules; put host-specific notes in `CLAUDE.md` (Claude Code).

> This file is the canonical project rules file for AI coding agents. `CLAUDE.md` imports it inline via `@AGENTS.md`, so Claude Code agents see the same rules. Edit this file for shared rules; put host-specific notes in `CLAUDE.md` (Claude Code).

## Project Intent

- This repo packages reusable Codex skills, installer scripts, templates, and automation for a local-first personal asset system.
- Keep canonical reusable logic in the repo.
- Keep user state outside the repo whenever possible.

## Source Of Truth

- Reusable skills live under `.agents/skills/`.
- Installer logic lives under `install/`.
- macOS launchd templates live under `ops/launchd/`.
- Runtime path resolution lives in `scripts/asset_runtime.py`.

## Optional Local Agent Overlay

- Developers may keep machine-local workflow preferences in `AGENTS.local.md` at the repo root.
- `AGENTS.local.md` is ignored by git and may be absent. Agents should read it when present, treat it as local preference for this checkout only, and never copy its contents into tracked files, docs, release artifacts, or package contents.
- Shared project rules still belong in `AGENTS.md` or repo docs; local overlay rules should not override privacy, packaging, or verification requirements.

## Development Harness Skills

- OpenRelix repo-maintenance harness skills may live under `.agents/skills/openrelix-*-harness/`.
- These harness skills are for developing this repository, not for the public npm install surface.
- Do not add development-only harness skills to `plugins/openrelix/` or the `package.json` `files` allowlist unless there is an explicit release decision.
- The npm package should continue to ship only the public reusable skill surface, currently `.agents/skills/memory-review/`, unless the package-surface decision is intentionally changed.

## Editing Rules

- Do not reintroduce hard-coded user paths like `/Users/<name>/...` into reusable scripts or docs.
- Do not commit user data, raw Codex history, generated reports, or launchd output logs.
- Never put personal information, internal-only project details, user-specific memory content, private paths, account names, tokens, logs, or proprietary snippets into files that may be published as open source. This includes source code, tests, docs, website assets, npm package contents, GitHub Pages content, release artifacts, screenshots, fixtures, generated examples, and changelogs.
- Keep personal or site-specific Codex native memory mappings outside the repo, for example in the external state root extension file. Open-source code may only contain generic parsers, generic fallbacks, and sanitized public examples.
- Prefer installer or template changes over one-off local setup instructions.
- When adding automation, make state roots and Codex home paths configurable through environment variables.

## Git / Merge Rules

- Feature development and bug fixes do not require a dedicated git worktree by default. Use the current checkout or a normal branch unless the task explicitly needs release isolation, dirty-state isolation, or parallel checkout isolation.
- When finishing development from a branch and preparing to merge remotely, merge through local `main` and target `origin/main`; do not merge into other remote branches such as `origin/xxx`. Exception: pushes to long-lived `bugfix/version_*` lines auto-backmerge to `main` via the Bugfix Backmerge workflow after its checks pass, so no manual merge is needed there.
- Long-lived version lines such as `bugfix/version_*` are never deleted. The delete-after-merge rule applies only to short-lived task branches (for example `claude/<task>` or `codex/<task>` pulled from a version line): after such a branch has been merged into both local `main` and `origin/main`, delete the local branch, and if a separate worktree was intentionally created for it and is clean, remove that worktree too.

## Commit Checks

- Before committing OpenRelix changes, run `python3 scripts/check_personal_info.py`, `git diff --check`, and focused tests for the touched code.
- Before release, publish, installer, docs/site, or package-surface changes, also run `python3 -m py_compile scripts/*.py install/*.py`, `python3 -m unittest discover -s tests`, and `npm pack --dry-run --json`.
- Treat these checks as the project rule even when local git hooks are not installed or are bypassed.

---
> Source: [openrelix/openrelix](https://github.com/openrelix/openrelix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
