## ai-getaway

> 不要做本地页面验证，只要构建通过就行，当前环境是Windows，请使用Windows相关的命令。

# Repository Guidelines

## Project Structure & Module Organization

不要做本地页面验证，只要构建通过就行，当前环境是Windows，请使用Windows相关的命令。

npm run build 因本机 C:\Users\63138\AppData\Roaming\npm\node_modules\npm\bin\npm-cli.js 缺失失败，用项目本地 Vite 入口完成等价构建验证。

This repository contains an 星空 AI with a Go backend at the root and a Vue frontend in `frontend/`.

- `main.go` starts the Gin API server.
- `config/`, `database/`, `router/`, and `middleware/` handle bootstrapping, persistence, routing, and request guards.
- `controller/`, `service/`, `model/`, `response/`, `upstream/`, and `utils/` hold handlers, business logic, models, response helpers, proxying, and utilities.
- `frontend/src/` contains Vue code in `api/`, `components/`, `stores/`, `App.vue`, `main.js`, and `style.css`.

## Build, Test, and Development Commands

- `go mod tidy`: sync Go dependencies after backend import changes.
- `go run .`: run the backend on `APP_PORT`.
- `go test ./...`: run backend unit tests.
- `cd frontend && npm install`: install frontend dependencies from `package-lock.json`.
- `cd frontend && npm run dev`: start Vite for local UI development.
- `cd frontend && npm run build`: create the production frontend bundle in `frontend/dist/`.
- `cd frontend && npm run preview`: preview the built frontend bundle.

Local development expects MariaDB and Redis. Create `ai_gateway` and copy `.env.example` to `.env` before running the backend.

## Coding Style & Naming Conventions

Format Go code with `gofmt`. Keep package names short, lowercase, and aligned with directories. Put request handling in `controller/`, models in `model/`, and cross-endpoint behavior in `service/`.

Frontend code uses Vue 3 single-file components and ES modules. Name components in PascalCase, such as `AdminPanel.vue`, and keep API wrappers in `frontend/src/api/client.js`.

## Testing Guidelines

Backend tests use Go's standard testing framework. Keep test files next to the package under test and name them `*_test.go`; examples include `middleware/api_key_test.go`, `service/quota_test.go`, and `upstream/proxy_test.go`.

No frontend test runner is configured. For UI changes, run `npm run build` and manually verify affected flows in Vite.

## Commit & Pull Request Guidelines

Recent history uses short imperative summaries, often `fix` or feature-oriented messages like `Add admin email notifications and template management`. Prefer a specific subject, for example `Fix quota reset handling`.

Pull requests should describe the user-visible change, list backend/frontend impact, note environment or migration needs, and include screenshots for UI changes. Link issues when available and mention checks run, such as `go test ./...` and `npm run build`.

## Security & Configuration Tips

Do not commit real `.env` secrets, upstream API keys, JWT secrets, database passwords, or Redis credentials. Keep `.env.example` current when adding configuration such as `PUBLIC_BASE_URL`, `ALLOWED_ORIGINS`, or upstream timeout behavior.

---
> Source: [alove77580/ai-getaway](https://github.com/alove77580/ai-getaway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
