## edge-telemetry-android

> Android telemetry library (AAR) published via JitPack as `com.github.NCG-Africa:edge_telemetry_android`. Provides automatic instrumentation for crash reporting, screen tracking, network monitoring, session management, and more — with minimal setup from consuming apps.

# Edge Telemetry Android SDK

## Overview

Android telemetry library (AAR) published via JitPack as `com.github.NCG-Africa:edge_telemetry_android`. Provides automatic instrumentation for crash reporting, screen tracking, network monitoring, session management, and more — with minimal setup from consuming apps.

## Build & Test

```bash
# Build the library
./gradlew :telemetry_library:assembleRelease

# Run unit tests
./gradlew :telemetry_library:testDebugUnitTest

# Run instrumented tests (requires emulator/device)
./gradlew :telemetry_library:connectedDebugAndroidTest

# Lint check
./gradlew :telemetry_library:lintDebug

# Publish locally (for testing)
./gradlew :telemetry_library:publishToMavenLocal
```

## SDK Versions & Toolchain

- **Min SDK:** 24 (Android 7.0)
- **Target SDK:** 34 | **Compile SDK:** 35
- **Kotlin:** 2.1.0 | **AGP:** 8.7.1
- **Java:** 11 (source + target)
- **Gradle:** 8.4+
- **Current version:** 2.1.13

## Project Structure

```
edge_telemetry_android/
├── telemetry_library/                    # Single library module
│   ├── build.gradle.kts                  # Library build config + JitPack publishing
│   └── src/main/java/com/androidtel/telemetry_library/
│       ├── EdgeTelemetry.kt              # Deprecated typealias → TelemetryManager
│       ├── compose/
│       │   └── EdgeTelemetryCompose.kt   # Compose-specific utilities
│       ├── testing/
│       │   └── EdgeTelemetryTester.kt    # Testing utilities for consumers
│       └── core/
│           ├── TelemetryManager.kt       # Main singleton entry point
│           ├── TelemetryConfig.kt        # Configuration data class
│           ├── TelemetryHttpClient.kt    # HTTP transport layer
│           ├── TelemetryInterceptor.kt   # OkHttp interceptor for network tracking
│           ├── OfflineBatchStorage.kt    # SharedPreferences offline persistence (200-envelope cap)
│           ├── ScreenTimingTracker.kt    # Screen duration measurement
│           ├── MemoryTracker.kt          # Memory pressure monitoring
│           ├── DeviceCapabilities.kt     # Runtime capability detection
│           ├── services/                 # Service-based architecture (Phase 2)
│           │   ├── EventTrackingService.kt
│           │   ├── SessionService.kt
│           │   ├── UserProfileService.kt
│           │   ├── CrashReportingService.kt
│           │   └── BatchProcessingService.kt
│           ├── models/                   # Data models (TelemetryBatch, TelemetryEvent, etc.)
│           ├── crash/                    # CrashReporter, CrashFingerprinter
│           ├── breadcrumbs/              # BreadcrumbManager (circular buffer, max 50)
│           ├── session/                  # SessionManager
│           ├── user/                     # UserProfileManager
│           ├── ids/                      # IdGenerator (device, session, user IDs)
│           ├── navigation/              # NavigationStackTracker
│           ├── device/                   # DeviceInfoCollector
│           ├── location/                # IpLocationProvider (opt-in)
│           ├── events/                  # JsonEventTracker
│           ├── validation/              # EventPayloadValidator, RuntimeEventValidator
│           ├── interceptors/            # ApiKeyRedactionInterceptor
│           ├── retry/                   # CrashRetryManager
│           └── payload/                 # FlutterCompatiblePayload (backend compat)
├── gradle/libs.versions.toml            # Version catalog
├── settings.gradle.kts                  # Single module: :telemetry_library
└── docs/                                # Documentation (migration guides, schemas, etc.)
```

## Architecture

Singleton `TelemetryManager` delegates to five services:

- **EventTrackingService** — custom events, metrics, network requests
- **SessionService** — session lifecycle, timeout (default 30min), statistics
- **UserProfileService** — user profile storage (SharedPreferences)
- **CrashReportingService** — uncaught exception handler, manual error tracking, breadcrumbs
- **BatchProcessingService** — event queue, batching (default 50), flush timer (default 30s), offline retry

Thread safety: `AtomicBoolean` for ready state, `ConcurrentLinkedQueue` for event buffer, `ReentrantReadWriteLock` in breadcrumb/navigation managers, `CoroutineScope(Dispatchers.IO)` for async work.

## Automatic Instrumentation (enabled by default)

These features work out of the box after `TelemetryManager.initialize()`:

| Feature | How it works |
|---|---|
| **Crash reporting** | Installs `UncaughtExceptionHandler`, captures breadcrumbs + stack trace |
| **Screen tracking** | `ActivityLifecycleCallbacks` auto-tracks Activity transitions |
| **Lifecycle tracking** | `ProcessLifecycleOwner` observer for app foreground/background |
| **Session management** | Auto-creates sessions, handles timeout + rotation |
| **Memory monitoring** | Tracks memory pressure levels and usage |
| **Storage monitoring** | Monitors device storage state |
| **Frame tracking** | Frame drop detection for performance |
| **Network tracking** | OkHttp interceptor (requires adding to client — see below) |

## Opt-in Features (disabled by default)

| Feature | Config flag |
|---|---|
| Location tracking | `enableLocationTracking = true` |

## Key APIs for Developers

### Initialization (required, typically in Application.onCreate)

```kotlin
val config = TelemetryConfig(
    apiKey = "edge_your_api_key",       // Must start with "edge_"
    endpoint = "https://edgetelemetry.ncgafrica.com/collector/telemetry"
)
TelemetryManager.initialize(application, config)
```

### Network Tracking (add interceptor to OkHttpClient)

```kotlin
val client = OkHttpClient.Builder()
    .addInterceptor(TelemetryManager.createNetworkInterceptor())
    .build()
```

### Custom Events & Metrics

```kotlin
val tm = TelemetryManager.getInstance()
tm.recordEvent("button_clicked", mapOf("button_id" to "checkout"))
tm.recordMetric("load_time_ms", 342.0, mapOf("screen" to "home"))
```

### Error Tracking (manual)

```kotlin
tm.trackError(exception, errorCode = "PAY_001", productId = "checkout", userAction = "submit_payment")
tm.recordCrash(throwable)
```

### Compose Navigation Tracking

```kotlin
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    TelemetryManager.getInstance().trackComposeScreens(navController)
    // ... NavHost setup
}
```

### User Profile

```kotlin
tm.setUserProfile(name = "John", email = "john@example.com", phone = "+254...")
tm.clearUserProfile()
```

### Breadcrumbs

```kotlin
tm.addBreadcrumb("User tapped settings", category = "user", level = "info")
```

### Session Control

```kotlin
tm.startNewSession()
val sessionId = tm.getSessionId()
```

### Context for Errors

```kotlin
tm.setProductContext("checkout_flow")
tm.setLastUserAction("submit_payment")
```

## Key Dependencies

- **OkHttp3** 4.12.0 — HTTP transport + network interceptor
- **Gson** 2.10.1 — JSON serialization
- **AndroidX Work** 2.9.0 — background retry scheduling
- **AndroidX Lifecycle** 2.9.2 — process/activity lifecycle observation
- **AndroidX Navigation** 2.9.3 — Compose navigation tracking
- **AndroidX Compose BOM** 2024.04.01 — Compose UI support

## Testing

- **Unit tests:** JUnit 4 + Mockk + Robolectric (`src/test/`)
- **Instrumented tests:** Espresso + Compose test (`src/androidTest/`)
- Test helper: `EdgeTelemetryTester` for consumer test setup
- `TelemetryManager.resetForTesting()` — internal method to reset singleton in tests

## Conventions

- API key validation: must start with `"edge_"` and be non-blank
- Endpoint validation: must be non-blank
- All events auto-enriched with: app info, device info, session info, user info
- Pre-init queue (max 50 items) buffers calls made before SDK is ready
- `EdgeTelemetry` is a deprecated typealias for `TelemetryManager`
- Payload format is Flask-compatible JSON matching the backend collector
- Permission required: `ACCESS_NETWORK_STATE` (declared in library manifest)
- Publishing: JitPack via `maven-publish` plugin, release variant with sources + javadoc

---
> Source: [NCG-Africa/edge_telemetry_android](https://github.com/NCG-Africa/edge_telemetry_android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
