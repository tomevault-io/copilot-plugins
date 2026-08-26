## openship-app

> Unofficial Android client for Openship (self-hostable deployment platform). KMP + Compose Multiplatform. MCP as primary API layer, SSE for real-time streams.

# Openship-App — GitHub Copilot Instructions

## Project
Unofficial Android client for Openship (self-hostable deployment platform). KMP + Compose Multiplatform. MCP as primary API layer, SSE for real-time streams.

## Tech Stack
- Kotlin 2.4.10, Compose Multiplatform 1.11.1, AGP 9.0.1
- compileSdk 36, minSdk 24, JDK 21
- Planned: Ktor 3.5.2, Kotlin MCP SDK 0.15.0, Koin 4.x, kotlinx.serialization 1.11.0
- Package: `com.kareemessam.openship` (app), `com.kareemessam.openship.shared` (shared)

## Architecture
- `shared/` (KMP): commonMain = networking (MCP + SSE), models, repository. androidMain = Keystore, OkHttp engine.
- `androidApp/` (Android): Compose UI, ViewModels, Koin DI, navigation.
- One shared Ktor HttpClient for both MCP transport and SSE streams.
- MCP for discrete ops (health, list projects). SSE for real-time (build logs, monitoring).

## Key Rules
- All networking code in `shared/commonMain/` — no Android imports in commonMain.
- Platform-specific code in `shared/androidMain/platform/`.
- Use `@Serializable` data classes with `Json { ignoreUnknownKeys = true }` (API has no versioning).
- SSE `log` event `data` field is **base64-encoded** — decode before display.
- Track `eventId`/`seq` for deploy log resume (`?since=<seq>` on reconnect).
- Don't hold MCP Client or SSE streams across backgrounding — reconnect on foreground.
- Monitoring SSE 404s in CLOUD_MODE — hide monitor tab for cloud instances.
- Use Ktor SSE plugin (NOT EventSource — can't set Authorization headers).
- Add dependencies to `gradle/libs.versions.toml` first, then reference in build.gradle.kts.

## API Endpoints (Base: {instanceUrl})
- `GET /api/health/env` — discovery, public, returns authMode
- `GET /api/projects/home` — list projects, PAT auth, tag `project:list`
- `GET /api/deployments/:id/stream` — SSE build logs, PAT auth, `?since=<seq>` for replay
- `GET /api/system/monitor/stream?serverId=<id>` — SSE server metrics, PAT auth, 3s interval

## Auth
- PAT: `Authorization: Bearer opsh_pat_<43-char base64url>`
- Modes: none (loopback), local (better-auth), cloud (PKCE OAuth)
- Store in EncryptedSharedPreferences (Android Keystore)

## Build
```bash
./gradlew :androidApp:assembleDebug    # build APK
./gradlew :androidApp:installDebug     # install on device
./gradlew :shared:allTests             # unit tests
```

## References
- Full API contract: README.md §7
- Openship server source: ../openship/
- MCP SDK: https://github.com/modelcontextprotocol/kotlin-sdk

---
> Source: [kareemessam09/Openship-App](https://github.com/kareemessam09/Openship-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
