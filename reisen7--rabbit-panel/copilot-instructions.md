## rabbit-panel

> ﻿# Repository Guidelines

﻿# Repository Guidelines

## Project Structure & Module Organization
`backend/` contains the Go application. Entry starts at `backend/main.go`; dependency wiring lives in `backend/config/config.go`; HTTP routes and handlers are in `backend/router/`; business logic is in `backend/service/`; storage adapters are in `backend/repository/`.  
`frontend/` contains the Vue 3 app. Views live in `frontend/src/views/`, shared UI in `frontend/src/components/`, API wrappers in `frontend/src/api/`, and Pinia stores in `frontend/src/stores/`.  
Frontend build output is written to `backend/.dist/` and served by the backend. Do not hand-edit `frontend/src/auto-imports.d.ts` or `frontend/src/components.d.ts`.

## Build, Test, and Development Commands
- `cd backend && go run main.go` runs the backend locally on `:3958`.
- `cd frontend && npm install && npm run dev` starts the Vite dev server on `:3000` with API proxying.
- `./rabbit.sh build` builds frontend and backend release artifacts.
- `./rabbit.sh build --skip-frontend` rebuilds backend only.
- `cd frontend && npm run build` produces the frontend bundle in `backend/.dist`.
- `cd frontend && npm run test` runs Vitest once.

## Coding Style & Naming Conventions
Use Go defaults: tabs for indentation, exported names in `PascalCase`, internal helpers in `camelCase`. Keep handlers thin and put logic in `service/`.  
In Vue/TypeScript, use 2-space indentation, `PascalCase.vue` for components, and `camelCase` for composables, stores, and utilities. Follow existing API naming such as `/api/system/*` and keep state management in Pinia stores rather than components.

## Testing Guidelines
Frontend tests use Vitest with `jsdom`. Test files must be placed under `frontend/tests/` and match `*.test.ts` or `*.spec.ts`; files outside that tree are ignored.  
Backend currently has no established test suite, so at minimum verify changes with `go build ./...` and a targeted manual smoke test. For UI changes, include a successful `npm run build`.

## Commit & Pull Request Guidelines
Recent history favors short, direct subjects in Chinese, for example `新增功能: AI 运维助手` or `重构后端模块边界`. Keep commit messages imperative and specific.  
PRs should include: purpose, scope, affected backend/frontend areas, test/build results, and screenshots for visible UI changes. Link related issues when applicable.

## Security & Configuration Tips
Production settings come from environment variables such as `PORT`, `JWT_SECRET`, `NODE_SECRET`, and update-related `RABBIT_*` variables. Never commit secrets.  
For Docker deployments, prefer `docker-compose.deploy.yml`; for binary deployments, remember frontend changes require rebuilding `backend/.dist` before rebuilding the backend binary.

---
> Source: [reisen7/rabbit-panel](https://github.com/reisen7/rabbit-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
