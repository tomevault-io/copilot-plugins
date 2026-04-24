## spacetradersio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SpaceTraders is a Kotlin Multiplatform (KMP) SDK and Android client for the [SpaceTraders IO](https://spacetraders.io) REST API MMO. The full OpenAPI spec lives at `../OpenAPISpec/space_trader_open_api_spec.json` relative to this directory and is the authoritative reference for all API endpoints, request/response shapes, and auth schemes.

**Targets:** The SDK targets Android (minSdk 28) and iOS (arm64 + simulator). The Android app is a standalone Jetpack Compose application. An iOS app will consume the SDK's xcframework natively. Desktop/web are out of scope.

## Git Branching Strategy

This project follows Git Flow:

- **`main`** — production-ready releases only. Never commit directly.
- **`develop`** — integration branch. All feature/bugfix work merges here first.
- **`feature/<name>`** — branched from `develop`, merged back to `develop` via PR.
- **`bugfix/<name>`** — branched from `develop`, merged back to `develop` via PR.
- **`release/<version>`** — branched from `develop` when cutting a release, merged into both `main` and `develop`.
- **`hotfix/<name>`** — branched from `main` for critical production fixes, merged into both `main` and `develop`.

```bash
# Start a new feature
git checkout develop && git pull
git checkout -b feature/my-feature

# Finish and push for PR
git push -u origin feature/my-feature
# Open PR targeting develop (GitHub default)
```

`develop` is the default branch on GitHub — all PRs target it automatically.

## Build Commands

All commands run from the `KMP/` directory. On Windows use `gradlew.bat` instead of `./gradlew`.

```bash
# Build Android debug APK
./gradlew :app:assembleDebug

# Compile SDK (catches type errors before running tests)
./gradlew :spacetradersiosdk:compileAndroidHostTestSources

# Run all SDK tests (androidHostTest + commonTest compiled for JVM)
./gradlew :spacetradersiosdk:allTests

# Run app unit tests (ViewModels, etc.)
./gradlew :app:testDebugUnitTest

# Run a single SDK test class
./gradlew :spacetradersiosdk:testAndroidHostTest --tests "com.brokenhuskysledteam.spacetradersio.sdk.MyTest"

# Run a single app test class
./gradlew :app:testDebugUnitTest --tests "com.brokenhuskysledteam.spacetradersio.ui.ships.ShipDetailViewModelTest"
```

iOS framework is built via `./gradlew :spacetradersiosdk:assembleSpacetradersiosdkKitReleaseXCFramework`. There is no iOS app in this repo yet — the SDK produces an xcframework (`spacetradersiosdkKit`) for consumption by a native Swift/Xcode project.

## Architecture

Two Gradle modules: `:spacetradersiosdk` (KMP shared library) and `:app` (Android entry point).

`:spacetradersiosdk` has three source sets:

- **`commonMain`** — all shared logic: API layer, domain models, use cases, data repositories. No UI code.
- **`androidMain`** — Platform actuals (`Platform.android.kt`) and Android Ktor engine (OkHttp).
- **`iosMain`** — Platform actuals (`Platform.ios.kt`) and iOS Ktor engine (Darwin).

`:app` is a standalone Android application using Jetpack Compose (not Compose Multiplatform). It owns all UI, ViewModels, navigation, and DI. It depends on `:spacetradersiosdk` for business logic.

The `expect/actual` mechanism in `Platform.kt` is the current example of platform-specific behaviour.

### App layer structure inside `app/src/main`

```
di/          <- Hilt module bridging SDK types into Android DI graph
navigation/  <- Navigation Compose routes (@Serializable objects) and NavHost
ui/auth/     <- Auth screen: register + token import (UDF: StateFlow + sealed events)
ui/dashboard/ <- Dashboard screen: agent info, logout (UDF: StateFlow + sealed events)
ui/components/ <- Reusable themed composables: TerminalCard, TerminalButton, TerminalTextField, TerminalProgressBar, ScanlineOverlay
ui/theme/    <- Retro terminal theme (dark-only, green-on-black, monospace, sharp corners)
ui/ships/    <- Ship list + detail screens (UDF: StateFlow + sealed events)
```

ViewModels use `Channel<NavigationTarget>` (not SharedFlow) for one-shot navigation to avoid re-delivery on config change.

The app uses a dark-only retro terminal aesthetic. Design inspiration images live at `../InsipirationImages/` (note: folder name has a typo). New UI should follow this visual language: green-on-black, bordered panels, monospace text, uppercase headers.

### Layer structure inside `spacetradersiosdk/src/commonMain`

```
api/        <- Ktor HttpClient, endpoint functions, request/response DTOs
api/mapper/ <- DTO-to-domain-model extension functions
domain/     <- Business models, use cases, repository interfaces
data/       <- Repository implementations
```

### Error handling

API errors are deserialized via `HttpCallValidator` in `SpaceTradersClient` and thrown as `SpaceTradersApiException(error: SpaceTradersError, httpStatus: Int)`. The sealed `SpaceTradersError` hierarchy maps all ~90 API error codes (source: `../OpenAPISpec/space_trader_api_error_codes.json`) into categories: `AuthError`, `NavigationError`, `ShipOperationError`, `ContractError`, `MarketError`, `ConstructionError`, `GeneralError`, plus standalone types and an `Unknown` fallback. App ViewModels catch `SpaceTradersApiException` and `when`-match on the sealed type.

## Key Dependencies & Versions

| Dependency | Version |
|---|---|
| Kotlin | 2.3.20 |
| Compose BOM (Jetpack, app only) | 2026.03.01 |
| AGP | 9.1.0 |
| androidx.lifecycle (app only) | 2.10.0 |
| Ktor | 3.4.2 |
| kotlinx-serialization | 1.10.0 |
| Napier (logging) | 2.7.1 |
| multiplatform-settings | 1.3.0 |

Ktor uses the `okhttp` engine for Android and `darwin` for iOS — both are already wired in `gradle/libs.versions.toml`.

## Testing

Tests live in `spacetradersiosdk/src/androidHostTest/` (JVM unit tests) and `spacetradersiosdk/src/androidDeviceTest/` (instrumented). Test dependencies: `kotlin.test`, `ktor-client-mock`, `kotlinx-coroutines-test`.

**Coverage goal:** 100% class, method, line, and branch coverage for all new code.

- **Pure unit tests** (mappers, enums): no extra setup needed beyond `kotlin.test`
- **Use case tests**: back `AccountsApi`/`ContractsApi` with Ktor `MockEngine`; use hand-written fakes for repository interfaces (no mockk); use `runTest` for suspend functions
- **MockEngine helper must be a `MockRequestHandleScope` extension** — `private fun MockRequestHandleScope.okJson(...) = respond(...)`, not a standalone function. `respond` is only available as an extension on that scope.
- **Test `HttpClient` must include `defaultRequest { contentType(ContentType.Application.Json) }`** — omitting it causes `Fail to prepare request body` because `setBody()` requires Content-Type, matching the production `SpaceTradersClient` setup
- **`buildMockSpaceTradersClient` must pass the token through** — the factory lambda must use `{ token -> ... }` not `{ _ -> ... }`, and append `Authorization: Bearer $token` when non-null. Ignoring the token means `authenticatedWith()` cannot be tested.
- **App ViewModel tests** (`app/src/test/`): use `kotlin-test-junit` bridge for JVM runner compatibility. Hand-written fakes implement SDK interfaces directly (no MockEngine needed). Use `StandardTestDispatcher` + `advanceUntilIdle()` for coroutine control, Turbine for `Channel`/`Flow` assertions.

## Gotchas

- `gradlew` must have the executable bit set in git (`git update-index --chmod=+x gradlew`). Windows does not preserve Unix permissions — omitting this causes CI to fail with exit code 126.
- The repo root IS the KMP project root. Git was initialized inside `KMP/`, so there is no `KMP/` subdirectory on CI runners or in the repo. Do not use `working-directory: KMP` in GitHub Actions workflows.
- `compileKotlinAndroid` is ambiguous in Gradle — always use `compileDebugKotlinAndroid`. To verify app compilation, use `:app:compileDebugSources` (not `:app:compileDebugKotlinAndroid` which does not exist as a task).
- SDK package namespace is `com.brokenhuskysledteam.spacetradersio.sdk.*` — all source and test files use this consistently.
- In non-KMP JVM modules (like `:app`), use `kotlin-test-junit` (not plain `kotlin-test`) to get `@BeforeTest`/`@AfterTest` annotations resolved. The plain artifact lacks the JVM-specific bridge.
- Material 3 `Shapes` slots require `CornerBasedShape` — use `RoundedCornerShape(0.dp)` for sharp corners, not `RectangleShape` (which is a generic `Shape` and won't compile).
- `assertIs<T>()` returns `T`, not `Unit`. Using it as an expression body (`fun test() = assertIs<Foo>(x)`) makes JUnit reject the test method as non-void. Use block bodies: `fun test() { assertIs<Foo>(x) }`.
- **Windows worktree removal may fail with "Filename too long"** — deep Gradle build output hits MAX_PATH (260 chars). Delete the directory manually then run `git worktree prune` to clean git's metadata.
- **`@Volatile` in `commonMain` requires `import kotlin.concurrent.Volatile`** — the annotation is not auto-imported. `synchronized {}` is JVM-only and unavailable in `commonMain`; use `@Volatile` + documented threading assumptions instead.
- **`EntityStateStore` has no `observeAll()`** — use `.entities: StateFlow<Map<K, T>>` directly when combining in a ViewModel.
- **`ProcessLifecycleOwner` requires `androidx-lifecycle-process`** — add `implementation(libs.androidx.lifecycle.process)` to `app/build.gradle.kts` and the corresponding entry to `gradle/libs.versions.toml` (uses the same `lifecycleRuntimeKtx` version ref).
- **`stateIn(WhileSubscribed)` + `StandardTestDispatcher`**: Without an active subscriber the upstream `combine` is never collected and `.value` stays at the initial value. Use `SharingStarted.Eagerly` in ViewModels if tests read `.value` directly. Even with `Eagerly`, synchronous `MutableStateFlow` changes (e.g. `_error.value = null`) still require `advanceUntilIdle()` before they propagate through `combine` to `uiState.value`.
- **`RefreshScheduler` test fixtures: always use far-future arrival times** — transit ships in tests must use `arrivalTime = "2099-01-01T01:00:00.000Z"` (not a past date). A past `expiresAt` makes `delay(0)` skip entirely and the action re-schedules itself immediately → infinite loop → OOM. Use `backgroundScope` (not `this`) for `RefreshScheduler(backgroundScope)` in `runTest` to avoid `UncompletedCoroutinesError`.
- **Default lambdas with multiple parameters**: `= {}` infers `() -> Unit` — for a `(A, B, C) -> Unit` default parameter, use `= { _, _, _ -> }` instead.
- **Sealed `when` exhaustiveness + composable navigation**: Adding a UI navigation event to a ViewModel's sealed event interface still requires a branch in the VM's `when` even if the VM doesn't handle it — use `is MyEvent.NavigationEvent -> Unit`.

## SpaceTraders API

- Base URL: `https://api.spacetraders.io/v2`
- Auth: Bearer token (JWT). Two token types: `AgentToken` (per-agent gameplay) and `AccountToken` (account management). Agent registration returns the AgentToken directly.
- `POST /register` requires `AccountToken` (not `AgentToken`) — the API validates the JWT `sub` claim and returns a clear error if the wrong type is sent.
- The OpenAPI spec (`../OpenAPISpec/space_trader_open_api_spec.json`) is the ground truth — always check it before implementing a new endpoint.
- Pagination is used extensively: requests accept `page` and `limit`, responses include a `meta` object with `total`, `page`, `limit`.
- **POST endpoints with no body must use `setBody("{}")`** — `SpaceTradersClient`'s `defaultRequest` sets `Content-Type: application/json` globally; sending an empty body with that header causes a 422 from the API. Always pass `setBody("{}")` for action endpoints (orbit, dock, etc.) even when the API spec shows no request body.

## Mobile MCP (Android Emulator Interaction)

The `mobile-mcp` MCP server enables live interaction with the running Android emulator.

- **Always use `mobile_list_elements_on_screen` for click targets** — never estimate coordinates from screenshots. Screenshots render at half native resolution (e.g. 720px wide) but element coordinates are in native pixel space (1080px). Guessing from screenshots will miss.
- **Workflow:** `mobile_list_elements_on_screen` -> get coordinates -> `mobile_click_on_screen_at_coordinates` -> `mobile_take_screenshot` to verify.
- **Always call `mobile_list_available_devices` first** — never assume a device ID; the connected device changes frequently.
- **Windows gotcha:** The MCP server command must use a `cmd /c` wrapper — `command: "cmd", args: ["/c", "npx", "@mobilenext/mobile-mcp@latest"]` in `.claude.json`. Plain `npx` is a `.cmd` script and cannot be spawned directly on Windows.
- MCP servers connect at session startup — config changes require a session restart to take effect.

## Gradle Config Notes

- Daemon JVM heap: 2 GB (`-Xmx2048m` in `gradle.properties`).
- `kotlin.code.style=official` is set in `gradle.properties`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarshallLadd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
