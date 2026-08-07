## legado-hub

> Repo-specific guidance for OpenCode sessions working in `legado-hub`.

# AGENTS.md

Repo-specific guidance for OpenCode sessions working in `legado-hub`.
Keep the root clean; see `docs/architecture/repository-layout.md` for the boundary rules.

## Stack

- **backend/**: FastAPI runtime with a public/reader entrypoint on `8765` and an admin entrypoint on `8766`. The deployment entrypoint is `python -m app.server`; `app.main:app` is the combined compatibility/test app. Run commands from `backend/`.
- **frontend/**: React 19 + Vite + shadcn/ui console. The same built dist is served on both ports, while `/api/auth/entrypoint` selects the reader-only or administrator login/UI surface. Vite proxies to `8765` by default; set `VITE_LEGADOHUB_ENTRYPOINT=admin` to proxy to `8766`.
- **plugins/**: source plugins loaded at runtime by `app/source_plugins/loader.py`, which recursively scans `plugins/sources/**/metadata.yaml`.

## First-time setup / dev run

Docker Compose with the published `xzixmn/legado-hub:latest` image is the recommended deployment path. `start.bat` is the Windows source-development bootstrap: it creates `.venv`, installs `backend/requirements.txt`, runs `python -m playwright install chromium`, builds the frontend, then starts uvicorn. Manual equivalent:

```
python -m venv .venv
.venv/Scripts/python.exe -m pip install -r backend/requirements.txt
.venv/Scripts/python.exe -m playwright install chromium
cd frontend && npm install && npm run build
cd ../backend && ../.venv/Scripts/python.exe -m app.server --host 0.0.0.0 --public-port 8765 --admin-port 8766
```

On first startup the lifespan handler initializes the SQLite DB at `backend/data/app.db`, creates `admin`, and prints a generated high-entropy password once when no password was configured. Explicit passwords are never echoed. The product supports only local or trusted-LAN operation; external tunneling and its network security are operator-owned and do not select an application mode.

## Commands

Backend (run from `backend/`):
- Run server: `python -m app.server --host 0.0.0.0 --public-port 8765 --admin-port 8766`
- Tests: `pytest` (config lives in `backend/pytest.ini`; `asyncio_mode = auto`)
- Single test: `pytest ../dev-assets/tests/test_shared_book_storage.py`
- Maintenance scripts: `python scripts/create_source_plugin.py`, `python scripts/validate_source_plugin.py` (these are the only scripts that belong in `backend/scripts/`; put probes/benchmarks in `dev-assets/`)

Frontend (run from `frontend/`):
- Dev server: `npm run dev`
- Build: `npm run build` (runs `tsc -b` then `vite build`)
- Lint: `npm run lint`
- Tests: `npx vitest` (jsdom; no `test` script defined; config is inside `vite.config.ts` via the `vitest/config` import)

There is no command-order contract beyond: build frontend before running the server in Docker/production, since the backend serves `frontend/dist`.

## Validation cadence

- Do not run the full test suite after every small code edit.
- For a small task, finish the complete scoped change first, then run the relevant tests once as a batch.
- For a large task, split the work into explicit phases and run the phase-relevant tests once at the end of each phase.
- Focused syntax checks or a single regression test are allowed while diagnosing a concrete failure; they do not replace the phase gate.
- Before any commit, push, release, or claim that implementation is ready to commit, run the canonical full verification once with `verify.ps1`.
- Docker, Compose, image, or deployment changes must also be accepted on the SSH host `本地测试` by default; preserve its bind-mounted runtime data and use an isolated temporary container for destructive smoke checks. A missing local Docker CLI is not a reason to skip this gate.
- Never claim a phase or task passed without reporting the actual commands and results from its scheduled validation gate.

## Tests are split between repo and local-only `dev-assets/`

This is the most likely thing to trip you up:

- `dev-assets/` is **gitignored**. Only a small allow-list of test files plus `dev-assets/tests/conftest.py` are committed (see the `!dev-assets/tests/...` block in `.gitignore`).
- `backend/pytest.ini` points `testpaths` to `../dev-assets/tests` and `--ignore`s many files there.
- `dev-assets/tests/conftest.py` *also* `pytest_ignore_collect`s a second set of files (live-acceptance, official-auth, source-plugin-fixture tests, etc.) because those depend on local-only assets.
- Net effect: a fresh checkout runs only the committed subset. Do not assume a test file you see referenced exists in the repo; do not rely on `dev-assets/tests/source_plugins/`, `official_auth/`, or any `test_*` listed in the ignore blocks.
- `dev_assets_test_loader.py` at the repo root contains a **hardcoded absolute path** `C:\Home\Workspace\UGit\legado-hub\...`; it breaks if the repo is relocated. Prefer importing test helpers through normal `pytest`/`pythonpath` rather than extending that loader.

## Plugin contract

- Each plugin is a directory under `plugins/sources/` (subdirs `official/`, `thirdparty/` are scanned recursively) containing `metadata.yaml` + `source.py`.
- `source.py` must export a `Source` class with **async** methods matching each `capability` declared in metadata (`search`, `detail`, `toc`, `chapter`, `chapter_reviews`, `explore`, `auth`; `explore` requires `explore_groups` + `explore`).
- `plugins/sources/official/` is gitignored except for the synchronized `qidian_com_web` runtime mirror, which is tracked so release builds can bundle it. Other official plugins, especially App variants, remain ignored.
- Docker images bundle `plugins/sources/official/qidian_com_web` and all third-party plugins. Other official plugins, especially App variants, remain outside the image.
- Default Compose bind-mounts `./plugins/sources/thirdparty` and `./plugins/sources/official` as writable override layers. At startup the entrypoint copies each bundled plugin directory only when that plugin ID is missing on the host; a host directory with the same ID is authoritative. This permits adding new sources and replacing individual versions without masking bundled plugins. The image starts the entrypoint as root with only the ownership and UID/GID transition capabilities, applies the non-zero `PUID`/`PGID` to mount roots, then `gosu` drops privileges before the app process. There is no Compose init sidecar. `docker-compose.plugins.yml` only selects an alternate host path.
- Official Qidian plugins are authored in the sibling repo `C:\Home\Workspace\UGit\QDFCCKK`; edit `source-plugin/WEB-plugin` or `source-plugin/APP-plugin` there first, then run `python sync-to-legado-hub.py --variant WEB-plugin` or `python sync-to-legado-hub.py --variant APP-plugin`. Do not hand-edit synced files under `plugins/sources/official/qidian_com_*` except for emergency inspection.
- Authoritative contract: `docs/architecture/source-plugin-contract.md` (+ `.zh-CN.md`). Template scaffold: `plugins/templates/source_plugin/`.
- Plugins must not own global concurrency/timeout/proxy/retry/cache/scheduling policy — those are backend runtime responsibilities.
- For writing new plugins, start at `docs/skills/book-source-craft/README.md`.

## Config and runtime data (mostly gitignored)

All paths centralized in `backend/app/config.py`:
- Unified runtime config: `backend/config/app_config.json`
- Per-plugin cookies: `backend/config/cookies/<plugin_id>.json` (host-store; legacy in-plugin `Cookie.json` is auto-migrated on startup)
- Runtime data under `backend/data/`: `app.db`, `browser_profiles/`, `novels/`, `lexicons/`, `cache/`, etc.
- Generated Legado aggregate output: `backend/generated/`

Default Docker bind mounts persist these paths at repository-root `data/`, `config/`, `generated/`, `runtime/`, `plugins/sources/thirdparty/`, and `plugins/sources/official/`. The latter two are writable override layers, with bundled plugins copied only for missing IDs. Compose passes non-zero `PUID`/`PGID` (defaults `1000`/`1000`) and `LEGADOHUB_CHOWN_DATA`; the entrypoint chowns small trees recursively, always repairs `data/app.db*`, and otherwise only chowns the `data` mount root unless `LEGADOHUB_CHOWN_DATA=1`. The app then runs as `PUID:PGID` via `gosu`. Both reader and admin ports bind `0.0.0.0`; when no fixed base URL is configured, requests from local/private IPv4 or valid IPv6 clients use their validated Host to generate Reading URLs. Fixed base URL deployments keep exact Host allow-lists.

Do not commit DBs, cookies, `app_config.json`, or anything under either the source runtime paths or repository-root Docker persistence directories.

## Browser / Playwright (Source Access Bridge)

Controlled by process environment when an override is needed:
- `LEGADOHUB_BROWSER_ENABLED` (1/0)
- `LEGADOHUB_BROWSER_PROVIDER`: `chromium` (embedded, requires `playwright install chromium`) or `browserless` (remote; set `LEGADOHUB_BROWSERLESS_WS` + optional `_TOKEN`)
- `LEGADOHUB_BROWSER_PUBLIC_BASE_URL`, `LEGADOHUB_BROWSER_PROFILE_ROOT`, `LEGADOHUB_BROWSER_CONNECT_TIMEOUT_MS`, `LEGADOHUB_BROWSER_ACTION_TIMEOUT_MS`

The release workflow builds `xzixmn/legado-hub` with a `node:22` frontend stage + `python:3.12-slim` runtime and `playwright install --with-deps chromium`. `docker-compose.yml` pulls that image, exposes reader port `8765` and admin port `8766` on all interfaces, and bind-mounts persistent backend state to the host. The LAN firewall must restrict management access. The repository does not ship a public reverse proxy or tunneling deployment.

## Conventions

- Default shell guidance here is PowerShell on Windows, but `start.bat` is a `.bat` entrypoint (CRLF, `@echo off`); don't rewrite it as PowerShell.
- Source files use LF; Windows batch uses CRLF.
- Frontend uses `@` alias to `frontend/src` (see `vite.config.ts`); ESLint allows `any` and disables `react-refresh/only-export-components` under `src/components/ui/**`.
- `.github/workflows/docker-build.yml` image channels:
  - push `main` → `beta` + `sha-<short>` only (dev/test; does **not** move `latest`)
  - push tag `v*` → `vX.Y.Z` + `latest` + `sha-<short>` (formal release)
  - Prefer LAN test hosts on `beta`; production on `vX.Y.Z` or `latest`
  - Bump root `VERSION` + `CHANGELOG.md` when cutting a formal tag
- Reading book-source rule markers (`backend/app/core/legado_source.py`):
  - **Beta / daily rule tests**: only bump `_READER_RULE_RELEASED_AT_MS` (wall-clock ms) so `lastUpdateTime` rises and Reading offers update. Do **not** change `_READER_RULE_VERSION` (display name).
  - **Formal release** (`v*` tag): bump both `_READER_RULE_VERSION` and `_READER_RULE_RELEASED_AT_MS`.
- Code verification remains local through `verify.ps1`.

---
> Source: [XziXmn/legado-hub](https://github.com/XziXmn/legado-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
