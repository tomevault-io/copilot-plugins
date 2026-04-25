## dockerberg

> Dockerberg is a single Docker container bundling Apache Iceberg (via Trino), SeaweedFS (S3-compatible storage), and PostgreSQL (JDBC catalog) for local development.

# CLAUDE.md

## Project Overview

Dockerberg is a single Docker container bundling Apache Iceberg (via Trino), SeaweedFS (S3-compatible storage), and PostgreSQL (JDBC catalog) for local development.

## Build & Test

```bash
make build        # Build the Docker image
make test         # Run all BATS tests (~14 min)
make test-fast    # Fast tests only (~6 min)
make test-slow    # Slow tests only (env_vars + persistence)
make clean        # Remove test containers, volumes, and image
```

Tests use [BATS](https://github.com/bats-core/bats-core) and live in `test/`. Common helpers are in `test/test_helper/common.bash`.

## Key Files

- `Dockerfile` — Multi-stage build: Ubuntu 24.04, PostgreSQL 16, SeaweedFS, Trino 476
- `entrypoint.sh` — Container startup script
- `supervisord.conf` — Process manager config for all services
- `trino/` — Trino config templates (envsubst'd at runtime)
- `Makefile` — Build and test targets
- `.github/workflows/ci.yml` — CI pipeline (test on PR, publish on tag/main)

## Git Conventions

Use [Conventional Commits](https://www.conventionalcommits.org/) for all commit messages:

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

Common types: `feat`, `fix`, `docs`, `ci`, `test`, `refactor`, `chore`, `build`.

Examples:
- `feat(trino): add memory configuration env var`
- `fix(entrypoint): wait for postgres before starting trino`
- `docs: update README with persistence instructions`
- `test: add BATS test for custom env vars`

## Style Notes

- Shell scripts use `bash`. Use `set -euo pipefail` in scripts.
- Trino configs are `.template` files processed by `envsubst`.
- BATS tests follow the existing pattern: one file per service/concern, using shared helpers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edpaget) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
