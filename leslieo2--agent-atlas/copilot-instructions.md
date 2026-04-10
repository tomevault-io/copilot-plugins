## agent-atlas

> This repository is organized as a monorepo with product-facing apps at the top level:

# Repository Guidelines

## Project Structure & Module Organization
This repository is organized as a monorepo with product-facing apps at the top level:

- `apps/control-plane/`: FastAPI service (`apps/control-plane/app`) and backend tests (`apps/control-plane/tests`).
  Control-plane architecture is feature-module based under `apps/control-plane/app/modules`; see `apps/control-plane/AGENTS.md` for backend-specific layering and dependency rules.
- `apps/web/`: Next.js App Router app with thin route entrypoints in `apps/web/app` and layered product code in `apps/web/src`.
  Frontend architecture follows `app -> widgets -> features -> entities -> shared`; see `apps/web/ARCHITECTURE.md` and `apps/web/AGENTS.md` for the current layering and ownership rules.
- `packages/`: shared contracts and SDK scaffolds for cross-plane reuse.
- `runtimes/`: runner scaffolds for framework-specific execution adapters.
- `docs/`, `infra/`, and `schemas/`: architecture docs, deployment assets, and neutral schema definitions.
- `apps/web/test`: Vitest + React Testing Library test files for UI and logic.
- `apps/control-plane/.venv`, `apps/control-plane/.uv_cache`, `apps/web/node_modules`, `apps/web/.next`, package/runtime `.venv` directories, and local CI artifacts such as `.phoenix/` or `apps/control-plane/.agent-atlas-*.db` are generated and should not be committed.

## Build, Test, and Development Commands
- Control-plane setup and checks (run in `apps/control-plane/`):
  - `make install` — sync runtime + dev dependencies for local development.
  - `make sync` — recreate the backend environment from `uv.lock` for reproducible CI/local bootstrap.
  - `make fmt` — run Ruff formatter and Python compile check.
  - `make lint` — run Ruff lint/format checks.
  - `make typecheck` — run mypy on `app/`.
  - `make test` — run unit tests plus non-workflow integration tests without the coverage gate for fast local verification.
  - `make test-workflow` — run long-running backend workflow tests for experiment, validation, and export loops.
  - `make test-check` — run pytest with coverage checks (`--cov-fail-under=70`).
  - `make security` — run Bandit scan on backend code.
  - `uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000` — start API locally.
- Web app setup and checks (run in `apps/web/`):
  - `npm install` — install JS dependencies.
  - `make install-ci` — install frontend dependencies with `npm ci` for reproducible CI runs.
  - `npm run dev` — run Next.js dev server on `http://localhost:3000`.
  - `npm run lint` — run ESLint.
  - `npm run typecheck` — run TypeScript strict check (`tsc --noEmit`).
  - `npm run test` — run Vitest suite.
  - `npm run build` — production build.
  - `make ci` — run the frontend CI bundle (`lint + typecheck + coverage + build`).
  - `npm run verify:full` — run the local full frontend verification bundle, including Playwright e2e.
- Root monorepo checks (run in repository root):
  - `make lint` — run repo-wide lint and syntax checks across apps, shared packages, and runtimes; local runs default to serial orchestration.
  - `make typecheck` — run all defined type checks; local runs default to serial orchestration.
  - `make test` — run app tests and shared package/runtime smoke checks; local runs default to serial orchestration.
  - `make build` — build the frontend and Python package distributions; local runs default to serial orchestration.
  - `make ci-packages` — validate shared packages and runtimes.
  - `make ci` — run full monorepo CI coverage across backend, frontend, shared packages, and runtimes; local runs default to serial orchestration.

## Coding Style & Naming Conventions
- Python: 4-space indentation, LF line endings, quote-style double, line length 100 (`ruff` config).
- TypeScript/React: 2-space indentation, strict TypeScript enabled, Prettier + ESLint formatting.
- Naming: Python uses `snake_case` for functions/modules/variables, `PascalCase` for classes; frontend uses `camelCase` for hooks/functions and `PascalCase` for React components/files.
- Keep API payload mapping explicit and deterministic; avoid implicit schema guessing.

## Testing Guidelines
- Backend tests use `pytest`; put backend tests under `apps/control-plane/tests` with `test_*.py`.
- Frontend tests use Vitest + React Testing Library; test files are `*.spec.ts`, `*.spec.tsx`, `*.test.ts`, `*.test.tsx` (covered by `apps/web/vitest.config.ts`).
- Frontend unit/integration tests live under `apps/web/test`, while Playwright e2e tests live under `apps/web/e2e`.
- Frontend coverage defaults to `app/**/*` and `src/**/*`.
- Generated test artifacts such as `__pycache__/`, `*.pyc`, coverage output, build output, `.next/`, and `node_modules/` are not part of the committed test structure.
- Run at least `make test` and `npm run test` before opening a PR.

## Commit & Pull Request Guidelines
The root and both subdirectories currently have no git history on `master`, so there is no local convention to infer. Use Conventional Commits now (`feat:`, `fix:`, `refactor:`, `test:`).
- PRs should include what changed, why, related issue/task link, and command outputs (`make test`, `npm run test`, `make lint` / `npm run lint`, `make typecheck` / `npm run typecheck`).
- For UI work, include relevant screenshots or short screen recordings.
- For API changes, include example request/response payloads and affected endpoints.

## Security & Configuration Tips
- Copy `apps/control-plane/.env.example` to `apps/control-plane/.env` and configure environment values before running.
- Set `NEXT_PUBLIC_API_BASE_URL` in frontend when backend URL is not `http://127.0.0.1:8000`.
- Do not commit secrets, local database dumps, or generated artifacts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leslieo2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/leslieo2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
