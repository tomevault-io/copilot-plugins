## codex-chat-transformer

> ﻿# CLAUDE.md — agent context for codex-provider-manager

﻿# CLAUDE.md — agent context for codex-provider-manager

Project: a Python (>=3.8) utility managing Codex Desktop provider profiles,
converting chats between providers, and syncing config/chats between machines.
Stdlib-only at runtime (Tkinter ships with CPython).

## Architecture (flat layout — each root .py is a top-level module)

- `codex_chat_transformer.py` — CLI (`codex-manager`): provider switching,
  config.toml edits, chat conversion, `main()` dispatches to `_cmd_*` handlers.
- `codex_manager_gui.py` — Tkinter GUI (`codex-manager-gui`); calls
  `_run_convert` for conversions.
- `chat_bridge.py` — Codex <-> Droid chat format translation (including
  system-reminder envelope filtering).
- `droid_provider_adapter.py` — imports Codex providers into the Droid/Factory
  provider store (`~/.factory`).
- `codex_sync.py` — sync server + client (HTTP, PIN-authenticated), session
  store (SQLite), file hash manifests.
- `sync_tray.py` — optional system tray wrapper (needs pystray/pillow).
- `_logging.py` — shared logging bootstrap (`CODEX_MANAGER_LOG`).

## Conventions & pitfalls

- **Tests**: one self-contained suite `test_smoke.py` with a custom runner.
  Run it directly (`python test_smoke.py`) — do NOT use pytest discovery.
- **Code style**: Ruff enforced (target py38, line length 100). Run
  `ruff check .` and `ruff format .` before finishing.
- **Logging**: user output on stdout via `print`; diagnostics via
  `_logging` (stderr, `CODEX_MANAGER_LOG`, `CODEX_MANAGER_LOG_FILE`).
- **Env vars**: `CODEX_HOME` (~/.codex), `FACTORY_HOME` (~/.factory),
  `CODEX_MANAGER_LOG`, `CODEX_SYNC_HOST/PORT`. Any new env var must be added
  to `.env.example`.
- **Compatibility floor**: must keep working on Python 3.8 — no match/case,
  no `str.removeprefix`, avoid new stdlib only available in 3.9+.
- **Secrets**: never commit API keys or real transcripts; fixtures use
  `@example.com`. The sync server requires a PIN for all mutating endpoints.
- **Windows**: repo is developed on Windows/PowerShell; keep paths
  `pathlib`-based. `codex_manager.cmd` launches the GUI.
- **CI**: `.github/workflows/tests.yml` runs `test_smoke.py` on 3.9/3.11/3.12
  across 3 OSes; keep it green.
- **Do not**: introduce runtime deps (stdlib-only), reorder existing
  `_cmd_*` dispatch, or change chat envelope filtering semantics without
  updating `test_smoke.py` tests covering it.

---
> Source: [lightcomc/codex-chat-transformer](https://github.com/lightcomc/codex-chat-transformer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
