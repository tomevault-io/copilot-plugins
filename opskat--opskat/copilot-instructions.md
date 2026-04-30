## opskat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpsKat is an AI-first desktop application for managing remote infrastructure (SSH, databases, Redis). Built with **Wails v2** (Go 1.25 backend + React 19 frontend). The desktop app communicates via Wails IPC — there is no HTTP API.

Module: `github.com/opskat/opskat`

## Common Commands

### Development
```bash
make dev              # Wails dev mode with hot reload
make install          # Install frontend deps (pnpm)
make run              # Run the embedded production build (no hot reload)
make clean            # Remove build/bin, frontend/dist, embedded opsctl, coverage files
```

### Build
```bash
make build            # Production build
make build-embed      # Production with embedded opsctl CLI
make build-cli        # Standalone opsctl CLI binary
make install-cli      # Install opsctl to GOPATH/bin
```

### Testing
```bash
make test                              # Go tests (internal, cmd/opsctl, cmd/devserver, pkg)
make test-cover                        # Coverage report → coverage.html, opens in browser
go test ./internal/ai/...             # Single package
go test ./internal/ai/ -run TestName  # Single test
cd frontend && pnpm test              # Frontend tests (vitest)
cd frontend && pnpm test:watch        # Frontend tests in watch mode
```

### Linting & Formatting
```bash
make lint             # golangci-lint (10m timeout, config in .golangci.yml)
make lint-fix         # golangci-lint with auto-fix
cd frontend && pnpm lint       # ESLint + Prettier
cd frontend && pnpm lint:fix   # ESLint auto-fix
```

### Extensions (DevServer)
```bash
make devserver EXT=<name>       # Run isolated dev server for one extension
                                # Builds extension in ../extensions, loads its WASM + manifest
make build-devserver-ui         # Rebuild embedded devserver UI (frontend/packages/devserver-ui)
```
DevServer refuses to start when `OPSKAT_ENV=production`. Extension source lives in a sibling repo at `../extensions/` (see reference memory).

### Plugin
```bash
make install-skill    # Register Claude Code opsctl plugin (symlinks plugin/ into ~/.claude)
```

## Architecture

### Backend (Go) — Layered Architecture

```
main.go (Wails entry)
  └─ internal/app/        App struct — Wails binding layer, all public methods exposed to frontend via IPC
       ├─ internal/service/    Business logic (15 service packages: ssh_svc, sftp_svc, ai_provider_svc, extension_svc, etc.)
       ├─ internal/repository/ Data access (12 repos with interface + impl pattern)
       └─ internal/model/      Domain entities
```

**Key subsystems:**
- `internal/ai/` — AI agent: provider abstraction (Anthropic/OpenAI), tool registry, command policy checker, conversation runner, context compression, audit logging
- `internal/sshpool/` — SSH connection pool with Unix socket proxy for opsctl CLI
- `internal/connpool/` — Database/Redis tunnel management
- `internal/approval/` — Inter-process approval workflow (Unix socket between desktop app and opsctl)
- `internal/bootstrap/` — App initialization: database, credentials, migrations, auth tokens
- `internal/embedded/` — Embedded opsctl binary (build tag: `embed_opsctl`)
- `pkg/extension/` — WASM extension runtime (wazero): manifest parsing, plugin lifecycle, host bridge (I/O, KV, file dialogs, action events), policy evaluation
- `internal/service/extension_svc/` + `internal/app/app_extension.go` + `app_ext_host.go` — extension install/load/wiring into the desktop app
- `cmd/opsctl/` — Standalone CLI tool for remote operations, designed for AI assistant integration
- `cmd/devserver/` — Standalone HTTP dev server for a single extension (loads one WASM + manifest, proxies frontend HMR)

**Repository pattern:** Each repo has an interface, a default singleton, and `Register()`/getter functions.

**Database:** GORM + SQLite, migrations in `/migrations/` using gormigrate.

**Credential encryption:** Argon2id KDF + AES-256-GCM, master key in OS keychain.

**Extension system:** Extensions are WASM modules loaded at runtime. Each extension declares tools in `manifest.json`. AI invokes extension tools via a **single `exec_tool` tool** (not individual tools per extension) — the handler at `internal/ai/tool_handler_ext.go` dispatches by `extension` + `tool` args, enforces the extension's policy type against asset policy groups, and calls `Plugin.CallTool`. Host capabilities exposed to WASM are defined by `HostProvider` in `pkg/extension/host.go` (I/O open/read/write, KV, asset config, file dialogs, logging, events).

### Frontend (React + TypeScript)

Located in `frontend/` — a pnpm workspace monorepo. Root app consumes `@opskat/ui` (from `packages/ui`); `packages/devserver-ui` is embedded by `cmd/devserver`. Uses Vite 6 bundler, Tailwind CSS 4, shadcn/ui (Radix), Zustand 5 for state.

**No React Router** — uses a custom tab-based navigation system (`tabStore`). Tab types: terminal, ai, query, page, info.

**State stores** (`src/stores/`): One Zustand store per domain — assetStore, tabStore, terminalStore, aiStore, queryStore, sftpStore, shortcutStore, terminalThemeStore.

**Backend calls:** Generated Wails bindings in `frontend/wailsjs/`. Import from `wailsjs/go/app/App`. Real-time updates via `EventsOn()`.

**i18n:** i18next with `zh-CN` and `en` locales in `src/i18n/locales/`.

**Terminal:** xterm.js 6 with split-pane support.

**Tests:** Vitest + happy-dom + React Testing Library. Setup file mocks Wails runtime at `src/__tests__/setup.ts`.

### CI (`.github/workflows/ci.yml`)

Runs on PR and pushes to main/develop:
- Go: golangci-lint + `go test`
- Frontend: `pnpm lint` + `pnpm test` + `pnpm build`

### Git Commit Convention

Use **gitmoji** for commit messages. Common prefixes:
- ✨ New feature
- 🐛 Bug fix
- ♻️ Refactor
- 🎨 UI improvement
- ⚡️ Performance
- 🔒 Security
- 🔧 Configuration / tooling
- ✅ Tests
- 📄 Documentation
- 🚀 Deploy / release related

### Conventions

- Go mocks: generated with `go.uber.org/mock` in `mock_*/` subdirectories
- Go test assertions: goconvey + testify
- Frontend formatting: Prettier (120 char width, 2-space indent)
- Soft deletes via Status field (StatusActive=1, StatusDeleted=2), not GORM soft delete
- i18n keys namespaced under `"common"` — use `t("key.subkey")`
- Version info embedded via ldflags at build time

---
> Source: [opskat/opskat](https://github.com/opskat/opskat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
