## xboard-client

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Environment

This is an Android application project called "byte_flow" built with Kotlin and targeting Android API level 35. The project follows standard Android development practices with Gradle as the build system.

## Common Development Commands

### Build Commands
```bash
# Build the project
./gradlew build

# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Clean build
./gradlew clean
```

### Testing Commands
```bash
# Run all tests
./gradlew test

# Run unit tests only
./gradlew testDebugUnitTest

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Run specific test class
./gradlew test --tests "com.byteflow.www.ExampleUnitTest"
```

### Development and Debugging
```bash
# Install debug APK to connected device
./gradlew installDebug

# Generate test coverage report
./gradlew jacocoTestReport

# Check for dependency updates
./gradlew dependencyUpdates
```

### Code Quality
```bash
# Run lint checks
./gradlew lint

# Generate lint report
./gradlew lintDebug
```

## Project Architecture

### Module Structure
- **app/**: Main application module containing all source code and resources
- **Package**: `com.byteflow.www` - follows reverse domain naming convention

### Technology Stack
- **Language**: Kotlin (targeting JVM 11)
- **Build System**: Gradle with Kotlin DSL
- **Min SDK**: 21 (Android 5.0)
- **Target SDK**: 35 (Android 15)
- **Dependencies**: Standard AndroidX libraries (Core KTX, AppCompat, Material Design)

### Key Configuration Files
- **build.gradle.kts**: Project-level and app-level build configurations
- **gradle/libs.versions.toml**: Centralized dependency version management using Gradle version catalogs
- **AndroidManifest.xml**: Application configuration and metadata

### Development Patterns
- Uses Gradle version catalogs for dependency management
- Follows Android recommended project structure
- ProGuard configuration available for release builds
- Standard Android testing setup with JUnit for unit tests and Espresso for UI tests

### Build Variants
- **Debug**: Development builds with debugging enabled
- **Release**: Production builds with potential code obfuscation (ProGuard configured but disabled by default)

## Important Notes

- This appears to be a fresh Android project with minimal custom code - only contains default template files
- No custom Activities, Fragments, or business logic implemented yet
- Uses modern Android development practices with AndroidX libraries
- Gradle wrapper is included for consistent build environment across different machines

## Development Instructions

- **Build Process**: 
  - Do not run build after writing code when working on Android Studio project

## AI Interaction Guidelines

- **Language Preferences**:
  - Always respond in Chinese (中文) when interacting with this project

---
> Source: [MagicNop/Xboard_Client](https://github.com/MagicNop/Xboard_Client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
