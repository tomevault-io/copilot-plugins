## thinmp-kmp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ThinMP_KMP is a Kotlin Multiplatform (KMP) music player app targeting Android and iOS, built with Compose Multiplatform for shared UI. The project is in early stages (scaffolded from KMP template).

## Build Commands

```bash
# Android
./gradlew assembleDebug          # Build Android debug APK
./gradlew installDebug            # Build and install on connected Android device/emulator

# iOS (requires Xcode)
# Open iosApp/iosApp.xcodeproj in Xcode and build from there
# Or build the shared framework:
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64

# Tests
./gradlew :composeApp:allTests           # Run all tests
./gradlew :composeApp:testDebugUnitTest  # Android unit tests only

# General
./gradlew clean
./gradlew build                   # Full build (all targets)
```

## Architecture

- **Single module project**: All shared code lives in `composeApp/`
- **Package**: `dev.tcode.thinmpk`
- **Source sets**:
  - `commonMain` — Shared Kotlin code and Compose UI (the bulk of the app)
  - `androidMain` — Android-specific implementations (e.g., `Platform.android.kt`, `MainActivity`)
  - `iosMain` — iOS-specific implementations (e.g., `Platform.ios.kt`, `MainViewController`)
  - `commonTest` — Shared tests
- **iOS entry point**: `iosApp/` directory contains the Xcode project that embeds the Compose framework
- **UI framework**: Compose Multiplatform with Material 3
- **Platform abstraction pattern**: `Platform.kt` defines the `expect` interface; `.android.kt` and `.ios.kt` provide `actual` implementations

## Key Config

- Kotlin 2.3.0, Compose Multiplatform 1.10.0
- Android: minSdk 24, targetSdk/compileSdk 36, JVM target 11
- iOS: arm64 + simulatorArm64, static framework named "ComposeApp"
- Version catalog: `gradle/libs.versions.toml`

---
> Source: [tcode-dev/ThinMP_KMP](https://github.com/tcode-dev/ThinMP_KMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
