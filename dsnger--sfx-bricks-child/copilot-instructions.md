## sfx-bricks-child

> Release workflow for SFX Bricks Child theme

# Release workflow (manual only)

## Permission and safety
- Always ask the user before any release action.
- Never run release steps automatically or without explicit approval.
- Release notes must be ASCII only (no emojis or Unicode symbols).

## Preferred path
- Use `./release.sh <version> [release_notes] [--rc]` from the theme root.
- The script owns versioning, changelog, tagging, and release publishing.

## Preflight checks
- Git working tree must be clean (the script enforces this).
- GitHub CLI (`gh`) must be installed and authenticated for non-RC releases.
- `build-theme.sh` must exist and be executable.

## What `release.sh` does
- Updates `style.css` version (unless re-releasing same RC).
- Prepends a new entry to `CHANGELOG.md`.
- Commits the version + changelog changes.
- Creates a git tag `vX.Y.Z`.
- Builds the theme zip via `build-theme.sh`.
- Creates a GitHub release and uploads the zip (skipped for RC).
- Cleans up the local zip after upload (skipped for RC).

## RC (release candidate) behavior
- Use `--rc` or include `_rc` in the version (e.g., `1.2.3_rc`).
- No tag push and no GitHub release; zip is kept locally.

## Critical warning
- On failure, `release.sh` runs `git reset --hard` and `git clean -fd`.
- Do not run the script if there are untracked files you want to keep.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dsnger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
