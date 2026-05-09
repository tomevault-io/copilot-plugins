## playpalace11

> - `server/` is the v11 game server (modern Python). Core modules live in `server/core/` (including `server/core/users/`, `server/core/tables/`, and `server/core/ui/`), game implementations in `server/games/`, shared game helpers in `server/game_utils/`, auth in `server/auth/`, persistence in `server/persistence/`, and localization in `server/messages/`.

# Repository Guidelines

## Project Structure & Module Organization
- `server/` is the v11 game server (modern Python). Core modules live in `server/core/` (including `server/core/users/`, `server/core/tables/`, and `server/core/ui/`), game implementations in `server/games/`, shared game helpers in `server/game_utils/`, auth in `server/auth/`, persistence in `server/persistence/`, and localization in `server/messages/`.
- `server/tests/` contains pytest test suites (unit, integration, and play tests).
- `clients/desktop/` hosts the wxPython desktop client (UI code under `clients/desktop/ui/`); `clients/web/` contains the browser client assets.
- `packaging/` centralizes container definitions and platform installers (Linux, Windows, etc.).
- `var/` stores runtime artifacts (SQLite DB, logs, packaged builds). Everything inside is gitignored except the directory marker.
- Design notes now live in `docs/design/`.

## Build, Test, and Development Commands
This repo uses Python 3.13+ and `uv` for dependency management.

- Server dev:
  - `cd server && uv sync` — install server deps.
  - `cd server && uv run python main.py` — run server on default `0.0.0.0:8000`.
  - `cd server && uv run python main.py --help` — view server flags (e.g., `--ssl-cert`, `--ssl-key`).
  - `cd server && uv run python tools/export_packet_schema.py` — regenerate the shared packet schema JSON (writes to both `server/` and `clients/desktop/`).
- Client dev:
  - `cd clients/desktop && uv sync` — install client deps.
  - `cd clients/desktop && uv run python client.py` — run client.
  - Helper launchers live under `scripts/` (e.g., `scripts/run_client.sh`, `scripts/linux-client`).
- Tests:
  - `cd server && uv run pytest` — run all server tests.
  - `cd server && uv run pytest -v` — verbose output.
  - Inside `nix develop .`, helper scripts are available from repo root:
    - `./scripts/nix_server_pytest.sh`
    - `./scripts/nix_client_pytest.sh` (installs pytest/pydantic into `.nix-python/` automatically)

## Packet Schema Workflow
- Authoritative packet definitions live in `server/network/packet_models.py`. Update these models whenever packet shapes change.
- Run `cd server && uv run python tools/export_packet_schema.py` after editing the models; commit the updated `server/packet_schema.json` and `clients/desktop/packet_schema.json`.
- Both the server and client perform runtime validation using these schemas, so stale JSON copies will break validation and should never be left un-regenerated.

## Coding Style & Naming Conventions
- Python uses 4-space indentation and standard PEP 8 naming: `snake_case` for functions/vars, `CapWords` for classes, `UPPER_SNAKE_CASE` for constants.
- Server state is largely dataclass-based; prefer explicit, imperative state changes in game logic.
- Follow existing patterns in `server/games/` and `server/game_utils/` rather than introducing new abstractions.
- Keep changes minimal and focused; reuse existing helpers and avoid over-engineering.
- Docstrings use Google-style format; keep them updated when behavior changes.

## Testing Guidelines
- Frameworks: `pytest` and `pytest-asyncio`.
- Naming: test files in `server/tests/` use `test_*.py` and test functions use `test_*`.
- Include play tests when adding new games or changing serialization/persistence logic.
## Localization Notes
- When adding or changing strings, update all locales in `server/locales/` (and `languages.ftl` if applicable).

## Commit & Pull Request Guidelines
- Recent commit messages are short, sentence-case descriptions (e.g., “Modernize server networking…”). Follow that style; keep it one line and descriptive.
- PRs should include: a clear summary, testing notes (commands + results), and links to relevant issues/threads if applicable.

## Security & Configuration Tips
- Use `wss://` in production by passing `--ssl-cert` and `--ssl-key`. Self-signed certs are OK for local testing.
- If you change networking or auth flows, update both server and client paths and add tests when possible.

---
> Source: [XGDevGroup/PlayPalace11](https://github.com/XGDevGroup/PlayPalace11) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
