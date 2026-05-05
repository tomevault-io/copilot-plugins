## bash-bootstrap-scripts

> Bootstrap and deploy script conventions for VM provisioning


# Bootstrap & Deploy Scripts

## Architecture overview

Bootstrap uses a **shared runner + per-stack config** pattern:

- **`scripts/homelab_bootstrap.py`** — `BootstrapRunner` class: unified flow (parse args → load env → validate → run stack steps → wire observability → validate compose → optional bring-up → print summary).
- **`scripts/homelab_logging.py`** — `StepTracker` class: numbered step progress (`[1/N] Step name...`) with verbose mode, warnings, and end-of-run summary.
- **`scripts/homelab_common.py`** — shared helpers: env loading (`load_env`), placeholder checks (`is_placeholder`), Docker checks (`require_docker`), observability config (`setup_observability_config`), VM identity (`resolve_vm_identity`), chown helpers, logging setup.
- **`docker_compose/<stack>/stack_config.py`** — per-stack config module consumed by `BootstrapRunner` and `deploy.py`.
- **`docker_compose/<stack>/bootstrap.py`** — thin wrapper (~20 lines): adds repo root to `sys.path`, imports `stack_config`, runs `BootstrapRunner(stack_config).run()`.
- **`deploy.py`** — top-level orchestrator: dynamically loads `stack_config` via `importlib.import_module()`, validates env, runs bootstrap, creates symlinks/shell helpers, runs `docker compose up -d`.

## stack_config.py contract

Each stack's `stack_config.py` must define:

| Attribute | Type | Purpose |
|-----------|------|---------|
| `STACK_NAME` | `str` | Human-readable stack name |
| `SCRIPT_DIR` | `Path` | `Path(__file__).resolve().parent` |
| `REQUIRED_VARS` | `list[str]` | Env vars that must be set (non-placeholder) |
| `COMPOSE_OVERLAYS` | `list[tuple[str, str, str, str]]` | `(env_var, filename, default, label)` tuples for optional compose files |
| `bootstrap_steps(env, config_base, args)` | returns `list[tuple[str, Callable]]` | `(step_name, fn(tracker))` pairs for stack-specific bootstrap steps |

Optional attributes: `REQUIRES_ROOT` (bool), `REQUIRES_DEBIAN` (bool), `REQUIRES_DOCKER` (bool), `SUPPORTS_UP` (bool), `COPIES_ENV_EXAMPLE` (bool), `POST_DEPLOY_ACTIONS` (list[str]), `post_deploy(env, config_base, tracker)`.

## Python script conventions

- **Shebang:** `#!/usr/bin/env python3`.
- **Stdlib only:** `argparse`, `logging`, `subprocess`, `pathlib`, `os`. No pip deps.
- **Package imports:** The `docker_compose/` tree and `scripts/` are Python packages (have `__init__.py`). Import shared code via `from scripts.homelab_common import ...` or `from scripts.homelab_bootstrap import ...` after adding repo root to `sys.path`.
- **Fail fast:** Use `raise SystemExit(1)` with a clear log message when prerequisites are missing.
- **Idempotency:** Re-running must be safe (create dir if not exists, skip mount if already mounted, don't overwrite config files that already exist).
- **No secrets in repo:** Do not commit `.env` or secrets.
- **Logging:** Use `log` from `homelab_common` for messages. Use `StepTracker` (from `homelab_logging`) for structured step-based progress output in bootstrap steps: `tracker.detail()` for verbose-only, `tracker.success()` / `tracker.warn()` / `tracker.fail()` for outcomes.

## Per-stack directory layout

```
docker_compose/<stack>/
  compose.yml          # Main stack definition
  compose.*.yml        # Optional overlays (recyclarr, bazarr, exporters, etc.)
  .env.example         # Required env vars (no secrets; copy to .env and fill)
  bootstrap.py         # Thin wrapper: BootstrapRunner(stack_config).run()
  stack_config.py      # Stack identity, required vars, overlays, bootstrap steps
  scripts/             # Per-stack helper scripts (setup_media_apps.py, gen_caddyfile.py, etc.)
    __init__.py
```

## Bootstrap flow (BootstrapRunner)

1. Optional re-exec as root (if `REQUIRES_ROOT`).
2. Optional Debian check (if `REQUIRES_DEBIAN`).
3. Parse common args (`--force`, `--non-interactive/-y`, `--up`, `--verbose/-v`).
4. Load and validate `.env` (required vars, placeholder checks).
5. Run stack-specific bootstrap steps (from `stack_config.bootstrap_steps()`).
6. Wire observability sidecars (when `ENABLE_OBSERVABILITY=1`).
7. Validate compose (`docker compose config`).
8. Optional bring-up (`--up` flag, only when not invoked by deploy).
9. Print summary (overlays, warnings, post-deploy actions).

## Shell scripts (remaining)

- **proxmox/scripts/** — Run on the Proxmox host; create VMs, attach Cloud-Init. Use `qm` and minimal deps.
- **docker_compose/core/update-caddyfile.sh** — Thin wrapper that calls `gen_caddyfile.py` and reloads Caddy.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
