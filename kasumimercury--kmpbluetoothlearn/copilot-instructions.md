## kmpbluetoothlearn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kotlin Multiplatform Bluetooth Proof of Concept (KMPBluetoothPoC) targeting Android and iOS. The project uses Compose Multiplatform for UI and implements Bluetooth LE scanning functionality on Android. iOS Bluetooth functionality is currently stubbed.

## Build and Development Commands

```bash
# Build the project
./gradlew build

# Build and install Android debug APK
./gradlew installDebug

# Run all tests
./gradlew test

# Run Android unit tests specifically
./gradlew testDebugUnitTest

# Run Android instrumented tests
./gradlew connectedAndroidTest

# Clean build
./gradlew clean

# Generate iOS framework
./gradlew linkDebugFrameworkIos*
```

## Architecture

### Platform Abstraction Pattern
- Uses Kotlin Multiplatform `expect/actual` mechanism
- `Platform` interface defines platform-specific behavior in `commonMain/kotlin/net/mercuryksm/Platform.kt`
- Platform implementations in `androidMain` and `iosMain` directories

### Bluetooth Provider Pattern
- `BluetoothProvider` interface abstracts Bluetooth functionality
- `AndroidBluetoothProvider` in `androidMain/kotlin/net/mercuryksm/Platform.android.kt` provides full BLE implementation
- `IOSBluetoothProvider` in `iosMain/kotlin/net/mercuryksm/Platform.ios.kt` is currently stubbed

### MVVM with Compose State
- `BluetoothViewModel` in `commonMain/kotlin/net/mercuryksm/BluetoothViewModel.kt` manages UI state
- Uses `mutableStateOf` for reactive state management
- UI in `commonMain/kotlin/net/mercuryksm/App.kt` with Material 3 design

## Key Implementation Details

### Android Context Management
- Uses singleton `ContextHolder` in Android platform code (marked as TODO for improvement)
- Context must be initialized in `MainActivity.onCreate()`

### Bluetooth Configuration
- Service UUID is hardcoded in `AndroidBluetoothProvider` (marked as TODO for configuration)
- 3-second scan timeout (also marked as TODO for configuration)
- Requires BLUETOOTH_SCAN and BLUETOOTH_CONNECT permissions

### Testing Structure
- Unit tests use MockK for mocking
- Some tests are currently broken and marked as TODO
- Test files in `commonTest` and `androidTest` directories

## Known Issues and TODOs

Several areas marked for improvement:
- Fix broken permission handling test in Android tests
- Replace singleton ContextHolder with proper dependency injection
- Make hardcoded Bluetooth service UUID configurable  
- Implement proper error handling with user feedback (snackbars/dialogs)
- Complete iOS Bluetooth implementation
- Fix async test execution issues with Handler-based code

## Project Structure

- `composeApp/src/commonMain/` - Shared multiplatform code
- `composeApp/src/androidMain/` - Android-specific implementations
- `composeApp/src/iosMain/` - iOS-specific implementations (mostly stubs)
- `composeApp/src/commonTest/` - Shared unit tests
- `composeApp/src/androidTest/` - Android integration tests
- `iosApp/` - iOS application wrapper

## Dependencies

Uses Kotlin 2.1.21, Compose Multiplatform 1.8.1, and AndroidX libraries. Version catalog in `gradle/libs.versions.toml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KasumiMercury) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
