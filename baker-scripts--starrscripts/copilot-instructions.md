## starrscripts

> Bash and Python utility scripts for Starr apps (Radarr, Sonarr, Lidarr, Readarr).

# StarrScripts

Bash and Python utility scripts for Starr apps (Radarr, Sonarr, Lidarr, Readarr).

## Structure
- `servarr/` — Shared Starr app utilities
- Root scripts — Individual automation tools (backup, dedup, update, etc.)

## Conventions
- Bash scripts use `.sh` extension
- Python scripts use `.py` extension
- All scripts should be POSIX-compatible where possible
- Use `.env.sample` as template for required environment variables

## Pre-commit
Pre-commit hooks configured via `.pre-commit-config.yaml`. Run `pre-commit run --all-files` before committing.

---
> Source: [baker-scripts/StarrScripts](https://github.com/baker-scripts/StarrScripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
