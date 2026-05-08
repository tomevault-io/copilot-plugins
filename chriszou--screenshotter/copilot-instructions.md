## screenshotter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Automated screenshot and activity tracking system with two components:
- **macOS app** (`macos_app/`) — Swift/SwiftUI menu bar app that captures screenshots, sends them to a vision AI for description, stores records in local SQLite, and syncs metadata to a remote server
- **Go server** (`screenshot-server/`) — REST API server that receives and stores activity metadata (no image files)

## Build & Run Commands

### macOS App
```bash
cd macos_app
swift build                    # Debug build
swift build -c release         # Release build
./build.sh                     # Release build + .app bundle + .dmg packaging
```
- Swift Package Manager, Swift tools 5.9, minimum macOS 13.0
- No external dependencies — uses only Apple system frameworks

### Go Server
```bash
cd screenshot-server
make build                     # Build binary
make run                       # Build and run (background, port 8080)
make dev                       # Run in development mode
make stop                      # Stop running server
make deploy                    # Deploy to VPS
make deps                      # Download Go dependencies
```
- Go 1.21, uses Gin + GORM + SQLite
- Docker: `docker-compose up -d`
- Env vars: `PORT`, `DATA_DIR`, `GIN_MODE`, `AUTH_TOKEN`

### Database Queries
Use `sqlite3` command-line tool to query local SQLite databases.

## Architecture

### macOS App (MVVM)
- `AutoScreenshotApp.swift` — Entry point, AppDelegate, NSStatusBar menu bar setup
- `ScreenshotViewModel.swift` — Core business logic: timer-based capture, AI dispatch, auto-cleanup. Auto-starts on init
- `ContentView.swift` — Main UI with control panel and scrollable history feed
- `ScreenshotRecord.swift` — Data model + `ScreenshotHistoryManager` singleton (in-memory cache)
- `DatabaseManager.swift` — SQLite3 via direct C API (singleton)
- `ServerSyncManager.swift` — HTTP sync to user-configured server (fire-and-forget)
- `KimiAPI.swift` / `SeedAPI.swift` — Two AI vision providers (Moonshot Kimi, Bytedance Seed 2.0 Lite), selectable via settings
- `ScreenAnalyzer.swift` — Local Vision OCR fallback (currently unused in main flow)

Key patterns:
- All classes use `@MainActor` isolation; background work via `Task {}` + `await MainActor.run {}`
- Singleton managers: `DatabaseManager.shared`, `ScreenshotHistoryManager.shared`, `ServerSyncManager.shared`
- Settings persisted in `UserDefaults` (API keys, interval, quality, AI provider)
- Local SQLite is source of truth; server sync is optional/resilient
- Screenshots auto-deleted 60 seconds after AI analysis (only metadata kept)
- Lock screen detected via `CGSessionCopyCurrentDictionary()`
- Local DB path: `~/Library/Application Support/AutoScreenshot/screenshots_v2.db`

### Go Server (MVC-lite)
- `main.go` — Gin router, CORS middleware, optional Bearer token auth
- `handlers/screenshot.go` — CRUD + batch endpoints under `/api/activities`
- `models/screenshot.go` — GORM model + request/response DTOs
- `database/database.go` — GORM + SQLite init with AutoMigrate

Key patterns:
- Upsert semantics on create (idempotent sync by UUID)
- Response DTOs intentionally slim (timestamp + activityDescription only)

## Notes
- No tests exist in either component
- No linter configuration; Swift uses `StrictConcurrency` experimental flag
- The README in `screenshot-server/` documents routes as `/api/screenshots` but the actual routes in `main.go` are `/api/activities`

---
> Source: [ChrisZou/screenshotter](https://github.com/ChrisZou/screenshotter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
