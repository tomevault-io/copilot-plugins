## semgrep-ghes-util

> semgrep-ghes-util is a CLI tool for syncing GitHub Enterprise Server (GHES) organizations to Semgrep SCM configs. It discovers GitHub orgs not yet onboarded to Semgrep and can create the necessary SCM configurations.

# CLAUDE.md

## Project Overview
semgrep-ghes-util is a CLI tool for syncing GitHub Enterprise Server (GHES) organizations to Semgrep SCM configs. It discovers GitHub orgs not yet onboarded to Semgrep and can create the necessary SCM configurations.

## Tech Stack
- Python 3.12+
- UV for package management
- argparse for CLI

## Project Structure
```
src/semgrep_ghes_util/
├── __init__.py
├── __main__.py      # Entry point for `python -m semgrep_ghes_util`
├── cli.py           # CLI argument parsing and command handlers
└── clients/
    ├── github_client.py   # GHES API client
    └── semgrep_client.py  # Semgrep API client
```

## Commands
```bash
# Install dependencies
uv sync

# Run CLI
uv run semgrep-ghes-util --help

# SCM commands (Semgrep SCM config operations)
uv run semgrep-ghes-util scm list-configs           # List all Semgrep SCM configs
uv run semgrep-ghes-util scm list-missing-configs   # List GHES orgs not in Semgrep
uv run semgrep-ghes-util scm create-missing-configs # Create configs for missing orgs

# GHES commands (GitHub Enterprise Server operations)
uv run semgrep-ghes-util ghes list-orgs             # List all GHES organizations
```

## Docker
```bash
# Build
docker build -t semgrep-ghes-util .

# Run with .env file
docker run --rm --env-file .env semgrep-ghes-util scm list-configs

# Run with individual env vars
docker run --rm \
  -e SEMGREP_APP_TOKEN \
  -e GHES_TOKEN \
  -e GHES_URL \
  semgrep-ghes-util scm list-configs
```

## Environment Variables
- `SEMGREP_APP_TOKEN` (required) - Semgrep API token
- `GHES_TOKEN` (required) - GitHub Enterprise Server token
- `GHES_URL` (optional) - GHES URL, can also be passed via `--ghes-url`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/r2c-CSE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
