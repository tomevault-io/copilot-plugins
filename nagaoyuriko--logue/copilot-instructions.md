## logue

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KMP (Kotlin Multiplatform) messaging app with a Rust backend. Users belong to exactly one server (no limit on server size). Servers are created/joined by entering a server ID — if the ID doesn't exist it's created, if it exists the user joins it. Authentication via OAuth (Apple/Google) with token-based sessions. Push notifications via FCM (Android + iOS).

Package name: `jp.co.pannacotta.kmp_example`

## Build & Run Commands

```bash
# Android app (requires JAVA_HOME set to Android Studio's JDK)
JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" ./gradlew :androidApp:assembleDebug
JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" ./gradlew :shared:build

# Shared framework for macOS (must rebuild after shared module changes)
JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" ./gradlew :shared:cleanLinkDebugFrameworkMacosArm64 :shared:linkDebugFrameworkMacosArm64

# Rust server (runs on 0.0.0.0:8080)
cd server && docker build -t logue-server .
docker run -d --name logue-server -p 8080:8080 logue-server
```

## Architecture

### Modules

- **`shared/`** — KMP module with business logic only (no UI). Targets: Android, iOS (x64/arm64/sim), macOS (x64/arm64). Uses `applyDefaultHierarchyTemplate()` so `appleMain` covers both iOS and macOS.
- **`androidApp/`** — Android app with Jetpack Compose UI. Depends on `shared`.
- **`iosApp/`** — iOS app with SwiftUI. Depends on `shared` via framework.
- **`macOSApp/`** — macOS app with SwiftUI. Depends on `shared` via framework.
- **`AppleShared/`** — Swift code shared between iOS and macOS (ViewModels, utilities, models).
- **`server/`** — Rust backend (Axum + SQLite + WebSocket). Database file: `messages.db`.

### Data Flow

1. App launch → auto-auth with saved token (`POST /auth`) or show login screen
2. Sign in → `POST /signin` with OAuth identity token → server creates/returns user (UUID-based user_id) + token
3. If nickname is not set → show nickname input screen → `POST /nickname?token=` to set it
4. If server_id is null → show server setup screen → `POST /servers/join?token=` to create/join server
5. Token saved in SharedPreferences (Android) / UserDefaults (Apple). All subsequent API calls use `?token=...`
6. Messages loaded via `GET /messages?token=...` (server-scoped), sent via `POST /messages?token=...` (broadcast to server)
7. Real-time updates via WebSocket (`/ws?token=...`) with auto-reconnect (3s delay)
8. Typing indicators sent/received via WebSocket (`typing_start` / `typing_stop` → `member_typing` with user_id)
9. Push notifications via FCM to all server members (except sender) on new message

### Naming Conventions (JSON API)

- **JSON (wire format)**: `snake_case` — Rust field names used as-is
- **Rust code**: `snake_case` (standard Rust convention, no serde rename needed)
- **Kotlin code**: `camelCase` (converted via `@SerialName("snake_case")`)
- **Swift code**: `camelCase` (via Kotlin shared module)

### Key Patterns

- **expect/actual**: `Platform.kt` (UUID), `ApiConfig.kt` (host), `HttpClientFactory.kt` (HTTP engine) — implementations in `androidMain` and `appleMain`
- **Repository pattern**: `MessageRepository` interface → `RealMessageRepository` (production) / `FakeMessageRepository` (test)
- **ViewModel + StateFlow**: `BoardViewModel` manages message list, input state, and typing indicator
- **OrNull pattern**: Kotlin suspend functions called from Swift use `*OrNull` variants that never throw, avoiding Kotlin/Native ObjC bridge crashes. Android uses the throwing variants directly.
- **CFlow**: Wrapper around Kotlin `Flow` for Swift observation. Primitive types (Boolean) must be wrapped as `Int` (0/1) for ObjC bridge compatibility.
- **Server model**: Users belong to exactly one server. `users.server_id` FK to `servers.server_id`. Messages are scoped to servers. WebSocket connections and push notifications are server-scoped.
- Android host: `10.0.2.2` (emulator), Apple host: `localhost`

### Server Endpoints

- `POST /signin` — provider + identity_token → user creation/lookup (UUID-based user_id) + token
- `POST /auth` — token validation (auto-login), returns user_id + server_id
- `POST /nickname?token=` — set nickname (1-20 chars)
- `POST /servers/join?token=` — join or create server by server_id
- `GET /messages?token=` — fetch server messages
- `POST /messages?token=` — send message + WebSocket broadcast to server + FCM push
- `POST /device-token?token=` — register FCM device token
- `GET /users?token=` — list server members with online status
- `GET /ws?token=` — WebSocket connection (message broadcast + typing indicators)

## Work Rules

- **No quick fixes**: Do not apply "just make it work" patches for bugs or inconsistencies. Identify the root cause and confirm the approach before fixing.
- **Confirm design decisions with the user**: When there are multiple options (naming conventions, API format, architecture changes, etc.), always present the options and ask for the user's decision.
- **Read before modify**: Do not propose or make changes to code you haven't read. Always read the relevant file first to understand the existing code before suggesting modifications.

## Tech Stack

- **Kotlin**: 2.2.10, **AGP**: 9.1.0, **Android SDK**: compile 36 / min 23 / target 36, Java 11
- **UI**: Jetpack Compose + Material 3 (via Compose Multiplatform plugin for dependency resolution)
- **Networking**: Ktor 3.1.1 (OkHttp on Android, Darwin on Apple)
- **Serialization**: kotlinx.serialization 1.8.0
- **Dependency management**: `gradle/libs.versions.toml` (Version Catalog)
- **Server**: Rust (Axum 0.8, SQLx/SQLite, bcrypt, tokio)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nagaoyuriko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
