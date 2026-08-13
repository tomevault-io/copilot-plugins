## codex-usage-tracker

> - `src/codex_usage_tracker/`: Python backend (ingestion, storage, CLI, reporting).

# Repository Guidelines

## Project Structure & Module Organization
- `src/codex_usage_tracker/`: Python backend (ingestion, storage, CLI, reporting).
- `ui/`: Next.js dashboard, API routes, and Playwright E2E tests.
- `tests/`: Python unit/integration tests (unittest style).
- `scripts/`: Packaging and install scripts (`package.sh`, `install.sh`).
- `docs/`: Project documentation.

## Build, Test, and Development Commands
- `python -m venv .venv && . .venv/bin/activate && python -m pip install -e .`: set up backend dev env.
- `pnpm --dir ui install`: install UI deps.
- `pnpm --dir ui dev`: run the Next.js dashboard locally.
- `codex-track web --no-open`: launch dashboard via CLI (auto-ingests rollouts).
- `./scripts/package.sh`: build the bundled UI + backend into `dist/`.
- `./scripts/install.sh`: install the packaged bundle and `codex-track` binary.

## Coding Style & Naming Conventions
- Python: 4-space indentation, PEP 8 naming (`snake_case` for functions/files).
- TypeScript/React: 2-space indentation, `camelCase` for variables, `PascalCase` for components.
- Prefer explicit, readable naming; avoid abbreviations in public APIs.
- Linting/typecheck: `pnpm --dir ui lint`, `pnpm --dir ui typecheck`.

## Testing Guidelines
- Python tests live in `tests/` and follow `test_*.py` naming.
- Run backend tests: `python -m unittest discover -s tests`.
- UI E2E tests live in `ui/tests/*.spec.ts` (Playwright).
- Run E2E tests: `pnpm --dir ui test:e2e`.
- E2E harness uses a generated fixture DB at `ui/tests/fixtures/usage.sqlite`.

## Commit & Pull Request Guidelines
- Commit messages are short, sentence-case summaries; optional conventional prefixes appear in history (e.g., `feat(overview): ...`).
- Keep commits focused and scoped to one logical change.
- PRs should include:
  - A clear summary of changes.
  - Testing notes (commands run and results).
  - Screenshots for UI changes when applicable.

## Security & Configuration Tips
- Local DB path is configurable via `CODEX_USAGE_DB`.
- UI reads config from the DB-adjacent `config.json` (timezone, pricing overrides).

## Agent-Specific Instructions
- For UI E2E, run exactly: `pnpm --dir ui test:e2e` so Codex uses the approved rule.

---
> Source: [CasperKristiansson/codex-usage-tracker](https://github.com/CasperKristiansson/codex-usage-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
