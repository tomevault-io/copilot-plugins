## mediaccc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MediaCCC is a Kotlin Multiplatform (KMP) app for browsing and watching talks from media.ccc.de and live streams from streaming.media.ccc.de. Built with Compose Multiplatform, targeting Android phones, Android TV, and iOS.

## Build Commands

```bash
# Android Phone
./gradlew :mobileApp:assembleDebug

# Android TV
./gradlew :tvApp:assembleDebug

# iOS — open iosApp/iosApp.xcodeproj in Xcode

# Run all tests
./gradlew allTests

# Run API unit tests (uses Ktor MockEngine)
./gradlew :api:testDebugUnitTest

# Run shared module tests
./gradlew :shared:testDebugUnitTest

# Run TV app tests
./gradlew :tvApp:testDebugUnitTest

# Lint
./gradlew lint
```

## Module Architecture

Four Gradle modules (`settings.gradle.kts`):

- **api/** — Ktor HTTP clients (`MediaCCCApi`, `StreamingApi`) with kotlinx.serialization models. Platform-specific HTTP engines: CIO (Android), Darwin (iOS).
- **shared/** — All shared Compose UI screens, ViewModels, Navigation 3 routes, Room KMP database, Koin DI modules, and repositories. This is the bulk of the app.
- **mobileApp/** — Android phone entry point. `MainActivity` with PiP support.
- **tvApp/** — Android TV entry point. Custom `AndroidTVPlayer`, TV-specific Compose screens with D-pad navigation, separate Koin modules.

## Key Architectural Patterns

**Data flow**: API clients → Repositories → ViewModels → Compose screens

- **Repositories** (`MediaRepository`, `StreamingRepository`, `PlaybackHistoryRepository`, `FavoritesRepository`, `QueueRepository`) wrap API clients and Room DAOs
- **ViewModels** live in `shared/src/commonMain/kotlin/com/jaeckel/mediaccc/viewmodel/`
- **DI** via Koin, configured in `shared/src/commonMain/kotlin/com/jaeckel/mediaccc/di/KoinModules.kt` with platform-specific modules in `PlatformModule.android.kt` / `PlatformModule.ios.kt`
- **Navigation** uses Navigation 3 (alpha). Routes defined in `shared/.../ui/navigation/NavRoutes.kt`, host in `AppNavHost.kt`
- **Room database** (`mediaccc.db`, version 3) with entities for playback history, favorites, and queue. Schemas in `shared/schemas/`.

## Platform-Specific Notes

- Android TV has its own player (`AndroidTVPlayer`), screens, and Koin modules in `tvApp/` — not in `shared/`
- iOS uses bundled SQLite for Room (`add("sqlite")` linking)
- `gradle.properties` sets `kotlin.native.cacheKind=none` as a workaround for Navigation 3 on iOS

## Testing

API tests use Ktor `MockEngine` with JSON fixtures in `api/src/androidHostTest/`. Koin DI verification tests exist in both `shared/` and `tvApp/`. Test dependencies: `kotlin.test`, JUnit 4, `ktor-client-mock`, `koin-test`, `kotlinx-coroutines-test`.

## Localization

Compose multiplatform string resources in `shared/src/commonMain/composeResources/values[-de|-es]/`. Three languages: English (default), German, Spanish.

## Version Catalog

All dependency versions managed in `gradle/libs.versions.toml`. Key versions: Kotlin 2.3.10, Compose Multiplatform 1.10.0, Ktor 3.4.0, Room 2.7.0, Koin 4.1.1.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/biafra23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
