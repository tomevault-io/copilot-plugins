## gocron

> A lightweight, distributed scheduled task management system written in Go with a Vue.js web interface.

# gocron

## Project Overview

A lightweight, distributed scheduled task management system written in Go with a Vue.js web interface.

## Tech Stack

- **Backend:** Go 1.26, Gin, GORM (MySQL/PostgreSQL/SQLite)
- **Frontend:** Vue 3 (Options API), Element Plus, vue-i18n, Vite, pnpm
- **RPC:** gRPC + Protocol Buffers
- **Auth:** JWT + TOTP 2FA

## Development

```bash
# Backend (with hot reload)
air

# Frontend dev server
cd web/vue && pnpm dev

# Build frontend
cd web/vue && pnpm build

# Run tests
go test ./...

# Build
go build ./...
```

## Project Structure

```
cmd/gocron/          - Main entry point
internal/
  models/            - GORM data models
  routers/           - Gin HTTP handlers (grouped by domain)
  service/           - Business logic (scheduler, execution)
  modules/           - Utilities (logger, i18n, notify, RPC)
web/vue/             - Vue.js frontend
  src/api/           - API client services
  src/pages/         - Page components
  src/components/    - Shared components
  src/locales/       - i18n (zh-CN, en-US)
  src/router/        - Vue Router config
  src/stores/        - Pinia stores
```

## Conventions

- Commit messages follow Conventional Commits: `feat:`, `fix:`, `chore:`, `refactor:`, `style:`, `test:`
- Do not add `Co-Authored-By` lines in commit messages
- Backend i18n: `internal/modules/i18n/zh_cn.go` and `en_us.go`
- Frontend i18n: `web/vue/src/locales/zh-CN.js` and `en-US.js`
- Database migrations: `internal/models/migration.go` (sequential version IDs)

---
> Source: [gocronx-team/gocron](https://github.com/gocronx-team/gocron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
