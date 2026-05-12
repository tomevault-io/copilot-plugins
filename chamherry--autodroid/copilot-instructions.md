## autodroid

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AutoDroid — Android automation platform. Kotlin adapter layer wraps Android system APIs, exposed via HTTP REST API (port 8080). React web frontend built to `assets/web/` and served as static files from the same HTTP server.

## Build & Test Commands

```bash
# Android APK (debug)
./gradlew :app:assembleDebug

# Automator unit tests (JUnit5 + MockK, 62 tests)
./gradlew :automator:test

# App unit tests (Router, Response, EventAdapter)
./gradlew :app:testDebugUnitTest

# Web frontend dev server (port 3000, proxies /api to localhost:8080)
cd web && npm run dev

# Web frontend build (output: app/src/main/assets/web/)
cd web && npm run build

# Full deploy: build web → build APK → install
cd web && npm run build && cd .. && ./gradlew :app:assembleDebug && adb install -r app/build/outputs/apk/debug/app-debug.apk

# TypeScript type check only
cd web && npx tsc --noEmit
```

## Architecture

```
Browser/Client → HTTP REST API (port 8080) → Controllers → Adapters → Android System APIs
                                            ↓
                                   StaticController → assets/web/ (React SPA)
```

**HTTP Server** (`server/`): Custom raw-socket server. Coroutine-per-connection (Dispatchers.IO + SupervisorJob). Express-like routing with `:param` support via regex. Middleware pipeline (CORS, Auth, Logger). 1MB max body, 30s timeout (SSE: no timeout + 30s heartbeat). Bind: 0.0.0.0 (all interfaces, requires token auth; constructor default 127.0.0.1 for testing).

**Security model**: Dual-scope token auth (READ for GET, FULL for POST/PUT/DELETE). 128-bit SecureRandom tokens in EncryptedSharedPreferences. Rate limiting on auth failures (5 attempts → exponential backoff). Shell exec endpoint is deliberately unrestricted (FULL token ≈ device root access) — this is by design for an automation tool. HTTP is plaintext; use on trusted networks only, or via `adb forward` for remote access.

**Route registration**: Each controller exports `registerXxxRoutes(server, adapter)`. All routes registered in `ApiRoutes.kt` via `registerAllRoutes()`. Static routes registered last as fallback (SPA).

**Response envelope**: All JSON responses wrapped as `{success: true/false, data/error, timestamp}`. Use `res.sendJson()`, `res.sendError()`, `res.sendBytes()`.

**Middleware pattern**: `fun interface Middleware { suspend fun handle(req, res, next) }` — call `next()` to continue pipeline.

**Adapter layer** (`adapter/`): 5 singleton adapters (Hilt `@Singleton`), aggregated in `AdapterContainer`. All public methods are `suspend` functions. Adapters wrap Android APIs; controllers never touch Android APIs directly.

**Automator engine** (`automator/`): Separate Gradle module. `UiObject` wraps `AccessibilityNodeInfo`. `SelectorParser` parses JSON selectors into `Filter` chains. `SearchAlgorithm` provides DFS/BFS traversal. `GlobalActionAutomator` dispatches gestures via `AccessibilityService.dispatchGesture()`.

**AccessibilityService**: Delegate pattern — `AutojsAccessibilityService` forwards events to registered `AccessibilityDelegate` instances. Flags: `FLAG_REPORT_VIEW_IDS | FLAG_RETRIEVE_INTERACTIVE_WINDOWS`. Multi-window dump via `service.windows`.

**DI**: Hilt. `AutojsNextApp` is `@HiltAndroidApp`. `AppModule` provides all adapter singletons. `AdapterContainer` groups them for route registration.

## Key Modules

- `app/src/main/kotlin/com/autodroid/server/` — HTTP server, router, middleware, 11 controllers
- `app/src/main/kotlin/com/autodroid/adapter/` — 5 adapters (Automator, App, Device, Shell, Event)
- `app/src/main/kotlin/com/autodroid/service/` — AccessibilityService, ScreenCaptureManager, ForegroundService
- `app/src/main/kotlin/com/autodroid/ui/` — Compose screens (legacy, being replaced by web/)
- `automator/src/main/kotlin/com/autodroid/automator/` — UiObject, Selector, Filter, Search, GlobalActionAutomator
- `web/src/` — React 19 frontend (Dashboard, UiInspector, Controls, Shell)
- `web/src/api.ts` — Typed HTTP API client

## HTTP REST API Endpoints

- `GET /api/status` — Server status (version, uptime)
- `POST /api/actions/*` — Coordinate actions (click, swipe, gesture, key)
- `GET /api/ui/dump` — Full accessibility tree (all windows, JSON)
- `POST /api/ui/*` — UI automation (find, click, input, scroll, wait)
- `POST /api/app/*` — App operations (launch, current, toast)
- `GET /api/device/info` — Device info
- `POST /api/shell/exec` — Shell commands
- `GET /api/screenshot` — Screen capture (JPEG via MediaProjection)
- `POST /api/screenshot/release` — Release capture resources
- `GET/POST /api/files/*` — File operations
- `GET /api/logs` — Log retrieval
- `GET /api/events/*` — Accessibility/key event streaming (SSE)
- `GET /*` — Static file serving (web frontend, SPA fallback)

## Tech Stack

- Kotlin 2.1.0, Coroutines 1.9.0, Hilt 2.53.1, KSP
- JUnit5 5.11.4, MockK 1.13.14
- React 19, Vite 8, TypeScript 5.9, xterm.js
- Min SDK 26, Target SDK 35, Java 17

## Package

`com.autodroid`

## Docs

- Architecture spec: `docs/specs/2026-03-23-autojs-next-architecture-spec.md`
- Development roadmap: `docs/plans/2026-03-23-roadmap.md`

---
> Source: [ChamHerry/AutoDroid](https://github.com/ChamHerry/AutoDroid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
