## freetodo

> - `lifetrace/`: FastAPI backend (routers, services, repositories, storage, llm, jobs, util).

# Repository Guidelines

## Project Structure & Module Organization
- `lifetrace/`: FastAPI backend (routers, services, repositories, storage, llm, jobs, util).
- `lifetrace/config/`: runtime configuration (`config.yaml` is generated from `default_config.yaml`).
- `lifetrace/data/`: runtime data (SQLite DB, vector DB, logs) — do not commit.
- `free-todo-frontend/`: Next.js + React frontend, Electron wrapper, and scripts.
- `.github/`: contribution docs, guidelines, and repository assets.

## Build, Test, and Development Commands
Backend (from repo root):
- `uv sync` — install Python deps into `.venv`.
- `python -m lifetrace.server` — start FastAPI server (auto-selects port from 8001).
- `uv run ruff check .` / `uv run ruff format .` — lint/format backend.

Frontend (from `free-todo-frontend/`):
- `pnpm install` — install frontend deps.
- `pnpm dev` — start Next.js dev server with auto-port detection.
- `pnpm lint` / `pnpm format` / `pnpm check` — Biome lint/format/check.
- `pnpm type-check` — run TypeScript type checks.

Packaging:
- `pnpm electron:build` (or `electron:build-win|mac|linux`) — build Electron app.
- `pnpm tauri:dev` / `pnpm tauri:build` — Tauri dev/build flows.

## Coding Style & Naming Conventions
- Python: PEP 8, type hints, docstrings; Ruff enforces 4-space indent and 100-char lines.
- TypeScript: Biome handles formatting/linting; keep components functional and hooks-safe.
- Naming: use Conventional Commits scopes like `backend`, `frontend`, `ui`, `config`.

## Testing Guidelines
- No dedicated test runner is configured yet. If you add tests, place Python tests under `tests/`
  and align with Ruff’s `tests/*.py` per-file ignores. For frontend, prefer a future `pnpm test`
  script; document any new runner in this file.

## Commit & Pull Request Guidelines
- Commit messages follow Conventional Commits:
  - Example: `feat(frontend): add calendar drag and drop`.
- In worktrees, prefer small, frequent commits. After each small feature change and
  after relevant checks pass, commit immediately. Only if all pending changes committed, notify the user
  (or other agents) that the commit is done.
- PRs should include: a clear description, linked issues (e.g., `Closes #123`),
  testing notes, and screenshots for UI changes. Use the `.github` PR template when available.

## Parallel Worktrees (Required for Concurrent Tasks)
- When working on multiple tasks in parallel, always use `git worktree` so each task
  has its own working directory, tests, and commits.
- Do not hardcode absolute paths or project names. Derive paths from the git repo root.
- Default worktree base directory is a sibling to the repo root:
  `<repo-parent>/_worktrees/<repo-name>/<task-slug>`.
- Keep the main worktree clean; each task uses its own branch and worktree.
- Each task must create a brand-new branch (do not reuse old branches).
- Branch naming must follow: `<Type>/<user>/<short-task>` where:
  - `Type` is lowercase: feat/chore/fix/hotfix/refactor (or other standard types).
  - `user` is the current git username (from git config).
  - `short-task` is a short summary (max 3 words).
- If a task name is provided, create a worktree first, then make changes in that worktree.
- Helper script (cross-platform): `python scripts/new_worktree.py "<task-name>"`
- Keep task branches in sync with the current mainline branch. Do not assume the
  mainline is named `main` or `master`, and do not assume the default remote is
  `origin`. Prefer syncing to a user-specified local mainline branch (e.g., `dev`,
  `dev-liji`, `dev-xxx`). If a mainline branch is not specified, ask the user
  which local branch to track. Only fall back to remote detection when needed,
  and detect the remote name first (e.g., `origin`, `upstream`).

## Worktree Dependency Sharing (Recommended)
- Install dependencies only once in the main worktree.
- Reuse them in other worktrees via linking scripts:
  - Windows (PowerShell): `powershell -ExecutionPolicy Bypass -File scripts/link_worktree_deps.ps1 -Main "<main-root>" -Worktree "<worktree-root>"`
  - macOS/Linux (bash): `scripts/link_worktree_deps.sh --main "<main-root>" --worktree "<worktree-root>"`
- Optional: add `--force` to overwrite existing links.

## Integration When Main Is Dirty
- Keep coding in task worktrees; do not commit on a dirty main worktree.
- Create a clean integration worktree and cherry-pick task commits into it.
- Run checks from the integration worktree (using shared dependencies), then merge.

## Security & Configuration Tips
- Do not commit `lifetrace/config/config.yaml` or `lifetrace/data/`.
- API keys and secrets should live in local configs or environment variables only.

---
> Source: [FreeU-group/FreeTodo](https://github.com/FreeU-group/FreeTodo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
