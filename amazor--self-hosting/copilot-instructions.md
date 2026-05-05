## docker-compose-stacks

> Docker Compose per-VM stack layout and conventions


# Docker Compose — Per-VM Stacks

The `docker_compose/` tree is a Python package (has `__init__.py`) with one sub-package per VM role. Each directory is self-contained: compose files, env example, bootstrap wrapper, and a `stack_config.py` module.

## Directory Layout (per VM)

```
docker_compose/<vm>/
  __init__.py        # Makes this a Python package
  compose.yml        # Main stack definition
  compose.*.yml      # Optional overlays (recyclarr, bazarr, exporters, etc.)
  .env.example       # Required env vars (no secrets; copy to .env and fill)
  bootstrap.py       # Thin wrapper: BootstrapRunner(stack_config).run()
  stack_config.py    # Stack identity, required vars, overlays, bootstrap steps
  scripts/           # Per-stack helper scripts
    __init__.py
```

Shared compose overlays live in `docker_compose/common/` (e.g. `compose.observability.yml`).

Active stacks: `core`, `monitoring`, `media`, `accelerated`. Future: `apps`.

## compose.yml

- Use the Compose V2 format (prefer current stable).
- One stack per VM; no cross-VM Compose dependencies. Cross-VM communication is via network (e.g. reverse proxy on core).
- Prefer explicit image tags over `latest` for reproducibility.
- Document any required env vars in comments or in `.env.example`.
- Optional overlays are controlled via `COMPOSE_OVERLAYS` in `stack_config.py` and enabled/disabled via env vars (e.g. `ENABLE_RECYCLARR=1`).

## .env.example

- List every variable the stack needs, with placeholder or example values.
- No real secrets. Use placeholders like `YOUR_TZ`, `YOUR_PUID`, or `CHANGE_ME`.
- `scripts/setup_env.py` can pre-fill auto-detectable values (random secrets, LAN IP, UID/GID).

## stack_config.py

The contract for each stack — consumed by both `BootstrapRunner` (via `bootstrap.py`) and `deploy.py` (via `importlib.import_module`). Defines `STACK_NAME`, `REQUIRED_VARS`, `COMPOSE_OVERLAYS`, `bootstrap_steps()`, and optional hooks (`post_deploy`, `POST_DEPLOY_ACTIONS`). See the bootstrap scripts rule for the full contract.

## bootstrap.py

Thin wrapper (~20 lines): adds repo root to `sys.path`, imports the stack's `stack_config`, and runs `BootstrapRunner(stack_config).run()`. All stack-specific logic lives in `stack_config.py`, not here.

## Storage & Boundaries

- Each VM mounts only what it needs; never mount "the whole NAS" everywhere.
- Mounts are scoped to subfolders/exports. `core` stays minimal and typically has no NFS mounts.
- NFS mounts are env-driven (host/export/mount-point from `.env`), managed by `ensure_nfs_mount()` in `homelab_bootstrap.py`.

## Naming and Consistency

- Folder names = VM hostnames (lowercase, short). Keep compose service names and env var names consistent with docs and README.

---
> Source: [amazor/Self-Hosting](https://github.com/amazor/Self-Hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
