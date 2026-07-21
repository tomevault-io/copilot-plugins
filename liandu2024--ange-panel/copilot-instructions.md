## ange-panel

> AnGe-Panel is a single-repository Go + Vue application.

# AGENTS.md

## Project Shape

AnGe-Panel is a single-repository Go + Vue application.

- Backend: Go module `sun-panel`, Gin, Gorm, SQLite/MySQL, sources at the repository root (`main.go`, `api/`, `router/`, `initialize/`, `models/`, `lib/`, `global/`, `structs/`).
- Frontend: Vue 3 + Vite + TypeScript + Pinia + Naive UI under `src/`.
- Built frontend output is served by the backend from `./web` at runtime. The committed `dist/` bundle is used by Docker and release packaging.
- Runtime mutable data lives in `conf/`, `database/`, `uploads/`, and `runtime/`; Docker maps these to `/data`.

## Important Compatibility Constraints

- Keep the Go module path as `sun-panel` unless doing a full repository-wide migration. Many imports and linker flags depend on it.
- Expect historical `Sun-Panel` naming in code, scripts, and UI metadata. Do not rename broadly unless the task explicitly calls for a branding migration.
- The current active project layout has the Go backend at the repository root. Some legacy scripts/workflows still reference `service/`; treat those references as stale unless you are intentionally fixing release automation.
- Do not remove or casually regenerate committed seed/runtime assets:
  - `seed/database/database.db`
  - `seed/uploads/**`
  - `dist/**`
  - executable `ange-panel`
- Do not overwrite user data paths (`database/`, `uploads/`, `conf/conf.ini`, `runtime/`) during development or migration work.
- `conf/conf.example.ini` is the source template for generated config; keep it in sync when adding config keys.

## Build And Verification Commands

Use the narrowest command that proves the change.

- Frontend install/type checks may be run with `pnpm install` and `pnpm run type-check`; frontend dev/build commands are intentionally blocked because the legacy `src/` styles do not match the current 3005 release UI.
- Frontend type check only: `pnpm run type-check`.
- Frontend lint: `pnpm run lint`.
- Backend compile: `go build ./main.go`.
- Backend package checks: `go test ./...` if tests exist or backend behavior changed.
- Docker image build expects `dist/` to already contain the frontend bundle: `docker build .`.

Notes:

- The frontend package manager is pnpm; avoid introducing new npm/yarn lockfile churn.
- `pnpm run build`, `pnpm run build:frontend`, `pnpm run build-only`, and `pnpm run dev` are blocked on purpose so the old Vite frontend cannot overwrite the current 3005 bundle.
- `add-frontend-version.js` mutates `.env` by updating `VITE_APP_VERSION`; be aware of that expected side effect.

## Runtime Behavior

- Backend startup is `initialize.InitApp()` then `router.InitRouters(":" + http_port)`.
- Default backend port comes from `[base] http_port` in `conf/conf.ini`/`conf.example.ini`, currently `3005`.
- Local development should use one visible port by default: the Go backend port from `conf/conf.ini` (currently `3005` in this workspace).
- Do not start the Vite dev server unless the user explicitly asks for frontend HMR.
- In this workspace, the current correct 3005 frontend styling is the backend-served `dist/` bundle. The checked-in `src/` frontend can produce an older-looking UI if rebuilt directly.
- Do not overwrite the current `dist/` bundle with `vite build`/`pnpm run build`. The package scripts intentionally block Vite dev/build; do not re-enable them unless the legacy frontend source/style mismatch has been resolved and the user explicitly asks for a source rebuild workflow.
- For urgent frontend fixes that must appear on 3005 while preserving the current visual style, patch the backend-served `dist/assets/**` file that `dist/index.html` actually references, then verify through port `3005`.
- After code changes, the working result must be visible through the backend-served app on port `3005`. Do not consider frontend work complete if it only works on a Vite development port.
- Keep backend-served frontend files cache-safe: `index.html`, `/assets/**`, `/custom/**`, and service-worker kill-switch files must not allow stale browser/PWA cache to keep an old 3005 UI alive after deployment.
- Vite dev server config exists only for deliberate frontend rebuild work. Because it creates a second origin with separate browser storage, it can look different from the backend-served app.
- Backend serves static frontend files from `./web` and uploaded files from `[base] source_path`, currently `./uploads`.
- SQLite is the default database. If the database file is absent, startup may seed from `seed/database/database.db`.
- Startup also seeds uploads from `seed/uploads` when `uploads/` is missing or effectively empty.

## API And Backend Conventions

- API routes are mounted under `/api`.
- Add backend endpoints by updating both the router package (`router/system`, `router/panel`, or `router/openness`) and the handler package (`api/api_v1/...`).
- Use the shared response helpers in `api/api_v1/common/apiReturn`; frontend expects JSON shaped like `{ code, msg, data? }`.
- Success code is `0`. Existing frontend auth/error behavior depends on codes such as `1000`, `1001`, and `1005`.
- Reuse middleware in `api/api_v1/middleware` for login/admin/public-mode access checks.
- Gorm uses singular table names. New persistent models must be added to `initialize/database/connect.go` `AutoMigrate`.
- Keep JSON field names compatible with existing TypeScript typings in `src/typings`.

## Frontend Conventions

- Use Vue 3 Composition API and existing project patterns.
- Use `@/` imports for `src/`.
- Add API wrappers under `src/api/**`; use `get`/`post` from `@/utils/request`.
- Request headers automatically include `token` and `lang`; do not duplicate that per call.
- Hard constraint: do not use browser `alert`, `confirm`, or `prompt` dialogs. Use the project's Naive UI message, notification, dialog, or modal patterns instead.
- Keep Pinia stores under `src/store/modules/**` and follow the existing `helper.ts` persistence pattern where relevant.
- Routing currently uses hash history (`createWebHashHistory`); do not switch history mode without backend/static serving changes.
- UI uses Naive UI, Tailwind utility classes, Iconify/local SVG icons, and existing components in `src/components`.
- Maintain i18n strings in both `src/locales/zh-CN.json` and `src/locales/en-US.json` for user-visible frontend text.
- Backend language strings live in `lang/zh-cn.ini` and `lang/en-us.ini`.

## Deployment Constraints

- Dockerfile builds the Go binary in the repository root and copies committed `dist/` into `/app/web`.
- Docker entrypoint initializes `/data/{conf,database,uploads,runtime}` and symlinks `/app/{conf,database,uploads,runtime}` to those persistent paths.
- GitHub release workflow verifies `dist/index.html`, copies `dist/*` to `web/`, builds `ange-panel`, and packages `conf/` plus `web/`.
- Docker deployment defaults to port `3005` and persists `/data/{conf,database,uploads,runtime}` for lossless upgrades.
- When a release version needs to change, increment the second decimal directly in order, for example `1.01` to `1.02` to `1.03`; do not create patch-style versions such as `1.00.1`.

## Change Discipline

- Prefer small, targeted changes that match existing naming and directory boundaries.
- Minimize every code change to the specific bug or request. Do not change unrelated UI styling, spacing, layout, behavior, versioning, release assets, or documentation while fixing a narrow issue.
- When a request names one exact UI property, change only that property. For example, if the request is to brighten button text in dark mode, do not change the button background, border, spacing, classes, markup structure, hover behavior, or other visual styling unless the user explicitly asks.
- Preserve backward compatibility for existing config files, database rows, uploaded file paths, and API response shapes.
- Before changing schema or seed behavior, consider both fresh installs and existing installations.
- Avoid broad formatting-only rewrites; the codebase contains mixed historical style and comments.
- Do not commit secrets, local database contents, generated logs, or local runtime uploads.

---
> Source: [liandu2024/AnGe-Panel](https://github.com/liandu2024/AnGe-Panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
