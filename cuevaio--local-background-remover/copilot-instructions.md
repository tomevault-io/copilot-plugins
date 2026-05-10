## local-background-remover

> - Root instructions for `/Users/cuevaio/projects/background-removal`.

# AGENTS.md

## Scope
- Root instructions for `/Users/cuevaio/projects/background-removal`.
- Local overrides exist in `apps/rmbg/AGENTS.md` and `apps/desktop/AGENTS.md`; `apps/web` uses this file.

## Workspace
- `apps/rmbg`: Python `rmbg` CLI. It owns model bootstrap, license enforcement, the worker server, and the release tarball built in GitHub Actions.
- `apps/desktop`: Electron app. The main process shells out to `rmbg`; the renderer only talks through `src/preload/preload.ts`.
- `apps/web`: Next 16 App Router site for marketing, checkout, license APIs, and installer/release routes. `/install` serves the public installer script.

## Setup
- Install JS deps with `bun install`.
- Sync CLI deps with `bun run py:sync` or `cd apps/rmbg && uv sync --group dev`.
- From the repo root, run CLI commands as `uv run --project apps/rmbg rmbg ...`; bare `uv run rmbg ...` can resolve a globally installed binary instead of this checkout.

## Commands
- Desktop dev: `bun run dev`
- Web dev: `bun run dev:web`
- `bun run lint` currently means Ruff for `apps/rmbg`, `tsc --noEmit` for `apps/desktop`, and a placeholder no-op for `apps/web`.
- `bun run test` currently means `pytest` for `apps/rmbg`, `bun test` for `apps/web`, and `bun run build && node --test test/*.test.cjs` for `apps/desktop`.
- `bun run check` only runs lint + test. It does not include the extra CLI syntax check from CI.
- `bun run build` hits every workspace; the web build is the real typecheck for `apps/web` and needs the server env from `.env.example`.

## Focused verification
- CLI single test: `cd apps/rmbg && uv run --group dev pytest tests/test_cli_license.py::test_worker_parser_accepts_idle_seconds -q`
- Desktop single test: `cd apps/desktop && bun run build && node --test test/rmbg-runtime.test.cjs`
- Web single test: `cd apps/web && bun test src/app/install/route.test.ts`
- CI order is `bun run lint` -> `bun run test` -> `cd apps/rmbg && uv run python -m py_compile rmbg_cli/cli.py`.
- After web changes, run `cd apps/web && bun run build` even if root lint/test pass.

## Gotchas
- Desktop runtime resolution order in `apps/desktop/src/main/rmbg-runtime.ts`: `RMBG_DESKTOP_CLI_PATH` -> packaged installed `~/.local/bin/rmbg` -> `apps/rmbg/.venv/bin/rmbg` -> `uv run --project apps/rmbg rmbg`.
- Packaged desktop installs the matching CLI version from `https://local.backgroundrm.com/install`; the installer smoke workflow verifies that exact URL and `~/.local/bin/rmbg`.
- Product kinds and license surfaces differ: checkout uses `app|cli|both`, but license enforcement uses `desktop|cli`; web normalizes `app` to `desktop`.
- Desktop processing depends on both `desktop` and `cli` entitlements. Keep that cross-surface behavior intact across web checkout/license code, Electron runtime code, and CLI license checks.
- CLI invariants: default model dir `~/.cache/background-removal/models/birefnet`, optional `BIREFNET_MODEL_DIR`, local/offline loading via `local_files_only=True`, exit codes `0/2/3/4/5/6`.
- Vercel Flags code throws unless `FLAGS` and `FLAGS_SECRET` are set. Local hint in code: `vercel env pull .env.local --yes`.
- For private repos, `/api/releases/latest` and `/releases/[tag]/[asset]` need `RMBG_GITHUB_TOKEN` or a manual `RMBG_LATEST_VERSION`.

## Release And Plans
- `v*` tags trigger `.github/workflows/release.yml`, which currently builds only the CLI tarball from `apps/rmbg` and rejects tag/version mismatches.
- Keep `.opencode/` tracked. Code commits are expected to include the matching `.opencode/plans/*` file.

---
> Source: [cuevaio/local-background-remover](https://github.com/cuevaio/local-background-remover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
