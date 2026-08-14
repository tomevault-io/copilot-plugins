## easypan

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Build & Test Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK (requires keystore.properties)
./gradlew assembleRelease

# Run unit tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.cook.easypan.easypan.data.repository.DefaultRecipeRepositoryTest"

# Run instrumented (UI) tests — requires a connected device or emulator
./gradlew connectedAndroidTest

# Run benchmark build
./gradlew :benchmark:connectedBenchmarkAndroidTest
```

## Project Architecture

MVVM with a strict 3-layer separation:

- **`core/`** — app-wide utilities: `CountdownTimer` (singleton flow-based timer),
  `SnackbarController` (singleton `Channel`-based event bus), shared composables (`EasyPanButtons`,
  `EasyPanText`), and `Result` sealed interface.
- **`easypan/data/`** — Firebase clients, DTOs, mappers, and repository implementations.
  `FirestoreClient` handles all Firestore reads/writes. `AuthClient` handles Google OAuth via
  `CredentialManager`. `AppSettings` (Proto DataStore serialized with `AppSettingsSerializer`)
  caches Firestore responses locally with TTL constants defined in `core/util/Constants.kt`.
- **`easypan/domain/`** — repository interfaces and domain models (`Recipe`, `User`, `UserData`,
  `RecipeStepDescription`).
- **`easypan/presentation/`** — one package per screen, each with `Screen.kt`, `ViewModel.kt`,
  `State.kt`, `Action.kt`, and a `components/` sub-package. All ViewModels expose a single
  `StateFlow<XState>` and accept a sealed `XAction`.
- **`di/AppModule.kt`** — single Koin module wiring all singletons and ViewModels. `FirebaseAuth`
  and `FirebaseFirestore` are provided as Koin singletons.
- **`services/`** — `CountdownTimerService` (foreground service for the cooking timer with WakeLock)
  and `FirebaseCloudMessaging`.

### Navigation

Three nav graphs layered in `RootNavGraph`:

- `AuthNavGraph` — shown when user is not signed in.
- `HomeNavGraph` (`Home`) — bottom nav with Home, Favorite, and Profile tabs.
- `SelectedRecipeViewModel` is a Koin-scoped shared ViewModel that carries the currently selected
  `Recipe` across `RecipeDetail → RecipeStep → RecipeFinish`.

### Caching

Repositories check `AppSettings` DataStore for cached data before hitting Firestore. Cache TTLs (1
hour for recipes/favorites, 20 min for user data) are in `Constants.kt`. The DataStore file is
`settings.json` via the `Context.dataStore` extension in `EasyPanApp.kt`.

### Signing

Release builds require `keystore.properties` at the project root with `storeFile`, `storePassword`,
`keyAlias`, `keyPassword`, and `clientId` (Google OAuth Web Client ID injected into
`BuildConfig.CLIENT_ID`).

### Testing

Unit tests use MockK. Tests for repositories construct a real Proto DataStore backed by a temp file
to test caching logic — do not mock the DataStore in these tests.

---
> Source: [DmytroTurchyn1/EasyPan](https://github.com/DmytroTurchyn1/EasyPan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
