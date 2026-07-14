## hermes-migrate

> End-to-end CLI migration tool: OpenClaw (`~/.openclaw/`) to Hermes (`~/.hermes/`). Stops OpenClaw, migrates everything (files, config, credentials), starts Hermes. Channels keep working after migration.

# CLAUDE.md - Project Guide

## What This Project Is

End-to-end CLI migration tool: OpenClaw (`~/.openclaw/`) to Hermes (`~/.hermes/`). Stops OpenClaw, migrates everything (files, config, credentials), starts Hermes. Channels keep working after migration.

## Architecture

```
hermes_migrate/
  __init__.py        # Package version
  cli.py             # argparse CLI entry point
  migrate.py         # All migration logic (~1700 lines)
tests/
  conftest.py        # Shared fixtures (sample configs, tmp dirs)
  test_cli.py        # CLI argument parsing tests
  test_migrate.py    # Migration logic tests (90+ tests)
  test_security.py   # Redaction and security tests
```

Entry point: `hermes_migrate.cli:main`

## Key Classes (migrate.py)

- `OpenClawMigrator` — orchestrates the 14-step pipeline
- `HermesInstaller` — detects/installs Hermes via official NousResearch installer
- `MigrationLogger` — console output with automatic secret redaction
- `MigrationResult` — dataclass tracking success/warnings/errors per step

## Migration Pipeline

1. `stop_openclaw()` — kill OpenClaw processes (pgrep + SIGTERM/SIGKILL)
2. `migrate_soul()` — SOUL.md persona file
3. `migrate_memory()` — MEMORY.md, USER.md, archive daily memories
4. `migrate_workspace_files()` — IDENTITY.md, AGENTS.md, TOOLS.md
5. `migrate_heartbeat()` — HEARTBEAT.md periodic tasks
6. `migrate_channels()` — map bindings to Hermes platform_toolsets in config.yaml
7. `migrate_models()` — primary model + fallbacks to config.yaml
8. `migrate_agents()` — document multi-agent setup in markdown
9. `migrate_advanced_config()` — compaction, concurrency, session, cron to config.yaml
10. `migrate_env_template()` — .env.openclaw with commented placeholders
11. `migrate_channel_details()` — per-channel account docs (redacted)
12. `migrate_infrastructure()` — gateway, hooks, plugins, providers docs
13. `migrate_credentials()` — real tokens/keys to ~/.hermes/.env (chmod 600)
14. `start_hermes()` — launch Hermes in background

## Key Design Decisions

- **Credentials ARE copied** to `.env` — the goal is working channels post-migration
- **Log output is redacted** — MigrationLogger scrubs tokens/keys from all console output
- **Documentation files are redacted** — `redact_sensitive_fields()` on all markdown reference docs
- **YAML fallback** — `_yaml_serialize()` / `_basic_yaml_load()` when pyyaml is missing
- **`_basic_yaml_load` returns `{}` for nested YAML** — can't parse it, so start fresh
- **`setdefault` pattern** for config.yaml to avoid overwriting existing Hermes values
- **All `open()` calls use `encoding='utf-8'`** for Windows compatibility
- **`SAFE_FIELD_ALLOWLIST`** prevents false positives (maxTokens, contextWindow matched "token")

## Development

```bash
pip install -e ".[dev]"
pytest -v                          # 107 tests
black hermes_migrate/ tests/   # format
ruff check .                       # lint
```

## CI/CD

- `.github/workflows/ci.yml` — tests on Python 3.9/3.11/3.12/3.13 x Linux/macOS/Windows
- `.github/workflows/publish.yml` — publish to PyPI on GitHub release (trusted publisher)

---
> Source: [raulvidis/hermes-migrate](https://github.com/raulvidis/hermes-migrate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
