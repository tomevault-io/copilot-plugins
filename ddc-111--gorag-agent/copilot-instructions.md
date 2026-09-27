## gorag-agent

> Executables live in `cmd/server/` and `cmd/indexer/`; domain and application code belongs under `internal/`, while reusable helpers live in `pkg/`. Backend tests are colocated as `*_test.go`. Runtime YAML is in `config/`, sample knowledge data in `data/knowledge/`, documentation in `docs/`, and deployment assets in `deployments/`.

# Repository Guidelines

## Project Structure & Module Organization

Executables live in `cmd/server/` and `cmd/indexer/`; domain and application code belongs under `internal/`, while reusable helpers live in `pkg/`. Backend tests are colocated as `*_test.go`. Runtime YAML is in `config/`, sample knowledge data in `data/knowledge/`, documentation in `docs/`, and deployment assets in `deployments/`.

The repository includes a Vue 3 admin app under `web/`. Put routed pages in `src/views/`, reusable UI in `src/components/`, feature code in `src/features/`, and static files in `public/`.

## Build, Test, and Development Commands

- `docker compose -f deployments/docker-compose.yml up -d`: start Redis and other local dependencies.
- `make run`: run the API with `APP_ENV=local`; on Windows, set the variable before `go run ./cmd/server`.
- `make build`: compile `bin/server` and `bin/indexer`.
- `make test` or `go test ./...`: run all backend tests.
- `make indexer`: import `data/knowledge/qa.csv` into the configured stores.
- `cd web && npm ci && npm run dev`: install dependencies and start Vite.
- `cd web && npm run build`: type-check Vue/TypeScript and produce the production bundle.

## Coding Style & Naming Conventions

Format Go with `gofmt`; use lowercase package names, exported `PascalCase` identifiers, and `camelCase` locals. Keep HTTP transport in `internal/server/handler`, workflows in `internal/application`, and persistence behind store interfaces.

For Vue and TypeScript, use two-space indentation. Name components and views `PascalCase.vue`, composables `useThing`, and modules with concise lowercase names such as `api.ts`.

## Testing Guidelines

Use Go's `testing` package and table-driven cases where inputs share behavior. Name tests `TestFunction_Scenario`. Cover changed handlers, middleware, stores, and error paths. No frontend test runner or coverage threshold is configured; require a clean TypeScript build and manually exercise changed UI flows.

## Commit & Pull Request Guidelines

Recent commits use short, feature-focused Chinese summaries. Keep each commit scoped and write an imperative summary in Chinese or English. Pull requests should explain behavior changes, list verification commands, link issues, and call out configuration or migration impacts. Include screenshots for UI changes and examples for API contract changes.

## Security & Configuration

Copy `.env.example` to `.env`; never commit API keys, JWT secrets, database files, or production credentials. Prefer environment overrides over editing shared YAML, and document any new variable in `.env.example`.

---
> Source: [ddc-111/gorag-agent](https://github.com/ddc-111/gorag-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
