## self-hosting

> This is a homelab IaC/documentation repo. There are no traditional package managers — Python scripts use only the standard library. The main tools are:

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is a homelab IaC/documentation repo. There are no traditional package managers — Python scripts use only the standard library. The main tools are:

- **Python 3** (stdlib only) — `deploy.py`, bootstrap framework, per-stack config modules
- **Docker + Docker Compose v2** — all services run as containers
- **`deploy.py`** — orchestrator: validates `.env`, runs bootstrap, creates symlinks/shell helpers, runs `docker compose up -d`

### Python architecture

The codebase is structured as importable Python packages (`__init__.py` files throughout):

- **`scripts/`** — shared framework (imported by all stacks):
  - `homelab_common.py` — env loading, placeholder checks, Docker checks, observability config, VM identity helpers
  - `homelab_bootstrap.py` — `BootstrapRunner` class (unified bootstrap flow), common compose/NFS/validation helpers
  - `homelab_logging.py` — `StepTracker` class (step-based progress output with verbose mode)
  - `setup_env.py` — post-clone helper that pre-fills `.env.example` with auto-detected values
- **`docker_compose/<stack>/`** — per-stack modules:
  - `stack_config.py` — stack identity, `REQUIRED_VARS`, `COMPOSE_OVERLAYS`, `bootstrap_steps()`, optional `post_deploy()`. This is the contract consumed by `BootstrapRunner` and `deploy.py`.
  - `bootstrap.py` — thin wrapper (~20 lines): adds repo root to `sys.path`, imports `stack_config`, runs `BootstrapRunner(stack_config).run()`.
  - `scripts/` — per-stack helper scripts (e.g. `setup_media_apps.py`, `gen_caddyfile.py`, `apply_authentik_blueprint.py`)
- **`deploy.py`** — top-level orchestrator: dynamically imports `stack_config` modules via `importlib.import_module()`, runs env validation, invokes bootstrap, creates symlinks/shell helpers, runs `docker compose up -d`.

### Key commands

| Task | Command |
|------|---------|
| Pre-fill `.env.example` | `python3 scripts/setup_env.py` |
| Deploy a stack | `python3 deploy.py <stack> --force --init-env -y` |
| Deploy all stacks | `python3 deploy.py all --force --init-env -y` |
| Validate compose | `docker compose -f docker_compose/<stack>/compose.yml config` |
| Lint Python | `python3 -m ruff check deploy.py scripts/ docker_compose/` |
| Compile-check | `python3 -m py_compile <file>` |

### Docker-in-Docker caveats (Cloud Agent sandbox)

- **node-exporter** container will fail to start due to missing host mount propagation (`rslave` on `/`). This is expected and does not affect other services.
- When running multiple stacks on the same host, **disable observability sidecars** on non-monitoring stacks to avoid container name conflicts (e.g. `ENABLE_OBSERVABILITY=0` in `.env`). In production, each stack runs on a separate VM so this is not an issue.
- Docker daemon must be started manually: `sudo dockerd &>/tmp/dockerd.log &` then wait a few seconds before using Docker commands.
- Use `--force` flag with `deploy.py` to skip `.env` placeholder validation (required in Cloud Agent since there are no real secrets/domains).

### Stack deploy workflow

1. `python3 scripts/setup_env.py` (pre-fills auto-detectable values)
2. `python3 deploy.py <stack> --force --init-env -y` (copies `.env.example` to `.env`, skips validation, runs bootstrap + compose up)

The `--init-env` flag auto-copies `.env.example` to `.env` when missing; `--force` skips placeholder guardrails; `-y` is non-interactive mode.

### Stacks available for testing

- **monitoring** — best for testing: self-contained, has Grafana UI on `:3000`, Prometheus on `:9090`, Loki on `:3100`, Uptime Kuma on `:3001`
- **core** — Caddy, Authentik (SSO), dnsmasq, whoami; needs `ENABLE_OBSERVABILITY=0` if monitoring is already running
- **media** — requires VPN credentials and NFS mounts; most complex to test locally
- **accelerated** — Plex (GPU transcoding), requires NFS mounts and Plex claim token

### No tests or CI

This repo has no automated test suite or CI/CD pipeline. Validation is done through:
- `py_compile` for syntax checking
- `ruff` for linting (E402 warnings are expected — scripts use `sys.path` manipulation before importing local modules)
- `docker compose config` for compose file validation
- `BootstrapRunner` validates `.env` and compose files as part of its workflow

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
