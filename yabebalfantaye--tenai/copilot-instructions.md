## tenai

> > Read by VS Code / GitHub Copilot when editing this repository.

# GitHub Copilot Instructions

> Read by VS Code / GitHub Copilot when editing this repository.

## Project: tenai-infra

Infrastructure-as-code for a Tailscale mesh network connecting Linux servers,
macOS, and Android into a unified development environment with AI agent tooling.

## Tech Stack

- **Shell**: Bash scripts (`set -euo pipefail`), cross-platform (Linux/macOS/Termux)
- **Python**: FastAPI webapp, Hydra config, managed by `uv` (not pip)
- **Config**: Hydra YAML (`config/defaults.yaml`), env vars (`.env`)
- **Orchestration**: GNU Make

## Conventions

- All install scripts are **idempotent** — use `command -v` guards before installing
- Config files use **marker-based blocks** (`# ── TENAI INFRA ... START/END ──`) for safe re-runs
- Python packages install via `uv pip install --system`, never `pip3`
- Platform detection via `scripts/detect.sh` → `$OS_TYPE` (linux/mac/termux)
- Tool selection via `INSTALL_ONLY` / `SKIP_TOOLS` env vars
- Docs in `docs/` follow prefix naming: `CONCEPT_`, `DATA_FLOW_`, `DEBUG_`, `EXAMPLE_`, `HOWTO_`, `MONITOR_`
- Secrets go in `.env` (gitignored), never in tracked files

## Key Directories

| Path | Purpose |
|------|---------|
| `scripts/install/` | Idempotent tool installers |
| `scripts/configure/` | SSH, aliases, git-ssh-setup |
| `scripts/conductor/` | Gemini session manager |
| `scripts/repos/` | Git worktree + agent dispatch |
| `webapp/` | FastAPI web control panel |
| `config/` | Hydra YAML device/repo configs |

## Common Patterns

```bash
# Guard pattern for installs
if command -v <tool> &>/dev/null; then
  echo "✓ <tool> already installed"
  return
fi

# Marker block pattern for configs
marker_start="# ── TENAI INFRA ... START ──"
marker_end="# ── TENAI INFRA ... END ──"
# Remove old block, write new block between markers

# Tool selection guard
should_install <tool_name> && install_<tool_name> || true
```

## Verification

After modifying Python code, always run:

```bash
make lint        # ruff linter
make test        # pytest suite (unit + integration)
```

Test files map to source files:
- `webapp/db.py` → `tests/test_db.py`
- `webapp/server.py` → `tests/test_api.py`
- `scripts/configure/generate_aliases.py` → `tests/test_aliases.py`
- `scripts/configure/resolve_host.py` → `tests/test_resolve_host.py`
- `scripts/env_loader.py` → `tests/test_env_loader.py`
- `config/*.yaml` → `tests/test_config.py`

---
> Source: [yabebalFantaye/tenai](https://github.com/yabebalFantaye/tenai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
