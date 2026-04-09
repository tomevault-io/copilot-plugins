## qlsv-kotlin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Android application called QLSV (Quản Lý Sinh Viên - Student Management System) built with:
- Kotlin
- Jetpack Compose for UI
- Hilt for dependency injection
- Firebase Authentication
- Navigation Compose for screen navigation

## Common Development Commands

### Build and Run
```bash
# Build the project
./gradlew build

# Clean build
./gradlew clean build

# Install debug APK on connected device/emulator
./gradlew installDebug

# Run the app on connected device/emulator
./gradlew installDebug && adb shell am start -n com.kma.qlsv/.MainActivity
```

### Testing
```bash
# Run unit tests
./gradlew test

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Run a specific test class
./gradlew test --tests "com.kma.qlsv.*TestClassName"
```

### Code Quality
```bash
# Run lint checks
./gradlew lint

# Generate lint report
./gradlew lintDebug
```

## Architecture

### Package Structure
- `com.kma.qlsv` - Root package containing App and MainActivity
  - `di/` - Dependency injection modules (Hilt)
  - `repository/` - Data layer with repository pattern
    - `auth/` - Authentication repository (Firebase Auth)
    - `admin/` - Admin-related repositories
    - `user/` - User-related repositories
  - `ui/` - Presentation layer
    - `navhost/` - Navigation setup
    - `screen/` - Individual screens (login, signup, home, intro)
    - `theme/` - Material3 theming

### Key Architectural Components

1. **Dependency Injection**: Uses Hilt with `@HiltAndroidApp` on Application class and `@AndroidEntryPoint` on activities
2. **Navigation**: Single-activity architecture with Navigation Compose managing screen transitions
3. **Repository Pattern**: Interfaces (IAuthRepository) with concrete implementations for data operations
4. **ViewModels**: Screen-specific ViewModels (LoginVM, SignupVM) for state management
5. **Firebase Integration**: Firebase Auth for user authentication, configured via google-services.json

### Important Files
- `app/build.gradle.kts` - App-level build configuration
- `gradle/libs.versions.toml` - Version catalog for dependencies
- `AppModule.kt` - Main Hilt module providing singleton dependencies
- `AppNavigation.kt` - Navigation graph definition

### Current Navigation Routes
- `"login"` - Login screen (start destination)
- `"register"` - Signup/registration screen  
- `"home"` - Home screen after authentication

## Development Notes

- Minimum SDK: 24 (Android 7.0)
- Target SDK: 35 (Android 15)
- Compile SDK: 35
- Java version: 11
- Compose BOM version: 2025.07.00
- Firebase services configured (requires google-services.json)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haroRT)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/haroRT)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
