## dev-workflow

> Local dev workflow — how to run, seed, test, and verify the stack


# Local Dev Workflow

## Daily development (default)

Use this for all feature work. Hot-reload, local Supabase, and seed scripts work.

```bash
pixi run start            # doctor + local Supabase + backend + frontend (from repo root)
```

Equivalent step-by-step: `pixi run doctor`, then `pixi run supabase` (or `pixi run supabase start`), then `pixi run dev`. Or skip doctor and run `pixi run supabase` then `pixi run dev` when the toolchain is already known good.

- Backend: uvicorn with `--reload` on http://localhost:8000
- Frontend: Vite dev server on http://localhost:3000
- DB: Supabase local Postgres on localhost:54322
- Tasks `backend`, `frontend`, and `dev` depend on `_is_supabase_running`, which starts local Supabase if it is not already up
- Seed local data (run once after first start, or after schema changes):

```bash
pixi run supabase reset                    # migrations + SQL seeds (org, departments, dev users)
pixi run import -- --vendors --products    # real Hike POS data
```

## Optional Docker stack

`docker compose up` from the repo root runs Redis + backend inside containers (see `docker-compose.yml`). Database is always Supabase (start local Supabase first with `pixi run supabase start`). Set `DATABASE_URL` in your `.env` to point at local Supabase (`postgresql://postgres:postgres@host.docker.internal:54322/postgres`). `devtools/` scripts are intentionally excluded from the Docker image — run seeds in native dev (`pixi run dev`) instead. Docker build context: `docker build -f backend/Dockerfile .`

## Key ports

| Service  | Native dev | Docker compose |
|----------|------------|----------------|
| Backend  | :8000      | :8000          |
| Frontend | :3000      | —              |
| Postgres | :54322     | :54322 (Supabase local)  |

## Common dev commands

Install [Pixi](https://pixi.sh) once (`curl -fsSL https://pixi.sh/install.sh | sh`). The binary lives in `~/.pixi/bin` — use a **new shell** or `export PATH="$HOME/.pixi/bin:$PATH"` until your `~/.zshrc` is sourced.

After cloning: `pixi install` (pulls Python 3.13, uv, Node 20, pnpm per [pixi.toml](pixi.toml) / [pixi.lock](pixi.lock)), then `pixi run doctor` to confirm nothing resolves from Homebrew/nvm/globals, then `pixi run uv sync --dev --directory backend` and `pixi run pnpm --dir frontend install --frozen-lockfile`.

All commands below run from the **project root** via `pixi run <task>` (`pixi task list` for discoverability). Do not use global `npm` / `npx` for this repo — use **pnpm** via `pixi run pnpm …` so the binary comes from the pixi env.

```bash
pixi run doctor                  # toolchain check (pixi python, uv, node, pnpm)
pixi run start                   # doctor + supabase start + dev (daily stack)
pixi run tests                    # all tests (backend then frontend then e2e)
pixi run tests backend -- …       # backend only; … = pytest args (one -- separates them from pixi task args)
pixi run tests frontend           # frontend tests only (vitest)
pixi run tests e2e -- …          # e2e only; … = extra args to Playwright after one --
pixi run lint [all|backend|frontend]    # default all (ruff + ESLint)
pixi run format [all|backend|frontend]  # default all (ruff + Prettier)
pixi run format-check [all|backend|frontend] # verify formatting without modifying (default all)
pixi run check [all|backend|frontend]   # lint then format (default all)
pixi run commit                  # commitizen conventional commit
pixi run backend                 # backend only
pixi run frontend                # frontend only
pixi run eval -- …               # extra args for evals
pixi run supabase                # start local Supabase stack (default action)
pixi run supabase stop           # stop local Supabase stack
pixi run supabase reset          # reset local DB from migrations + seeds
pixi run supabase typegen        # regenerate SQLModel types from local Supabase schema
pixi run clean                   # wipe backend venv + Python caches (see devtools/scripts/clean_repo.sh)
```

Never suggest `pip`, `python -m pytest`, or `ruff` directly. Python tooling runs through **`uv`** from the **pixi** environment, invoked as **`pixi run uv run --directory backend ...`** or via pixi tasks. After `pixi run clean`, recreate the venv with `pixi run uv sync --dev --directory backend` and re-run `pixi run doctor`. Production and dev dependencies, plus Ruff/pytest/commitizen config, live in **`backend/pyproject.toml`**; **`backend/uv.lock`** and **`backend/.venv`** are the uv project artifacts (`backend/.venv` must reference the interpreter under **`.pixi/envs/default/`**).

## Monorepo layout

```
sku-ops/
├── backend/          # deployable app + its tests + pyproject.toml + uv.lock
│   ├── pyproject.toml # Python deps (prod + dev), ruff, pytest, commitizen
│   ├── uv.lock
│   └── tests/        # backend unit/integration/api tests
├── frontend/         # React SPA (co-located unit tests in src/__tests__/)
├── e2e/              # cross-stack Playwright tests
├── devtools/         # seeds, evals, scripts — not deployed
└── pixi.toml         # task runner (dev, tests, supabase, …)
```

Each deployable owns its tests. Devtools live at the workspace root. The Docker image contains zero test or devtools code — `backend/tests/` is excluded via `.dockerignore`, devtools is structurally outside the `COPY backend/` layer, and `uv sync --no-dev --directory backend` skips test deps.

## Frontend always proxies /api → localhost:8000

Configured in `frontend/vite.config.js`. Works for native dev as long as the backend is on :8000.

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
