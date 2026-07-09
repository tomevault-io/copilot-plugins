## telephotos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

This is a standard Android project using Gradle and Kotlin.

- **Build APK:** `./gradlew assembleDebug`
- **Run Unit Tests:** `./gradlew testDebugUnitTest`
- **Run a Single Unit Test:** `./gradlew testDebugUnitTest --tests "com.tes.telephotos.ExampleUnitTest"`
- **Run Instrumented Tests:** `./gradlew connectedDebugAndroidTest`
- **Lint:** `./gradlew lintDebug`
- **Clean:** `./gradlew clean`

## Architecture and Structure

- **Language:** Kotlin 2.0+
- **Build System:** Gradle (Kotlin DSL `build.gradle.kts`)
- **Min SDK:** 24
- **Target SDK:** 35
- **UI Framework:** Standard Android Views/XML (not Compose). The main entry point is `MainActivity.kt` with `activity_main.xml`.
- **Package:** `com.tes.telephotos`

### Project Layout

- `app/src/main/java/`: Main Kotlin source code.
- `app/src/main/res/`: Android resources (layouts, drawables, values).
- `app/src/test/`: Local unit tests (JUnit).
- `app/src/androidTest/`: Instrumented Android tests (Espresso).
- `gradle/libs.versions.toml`: Centralized dependency management using version catalogs.

---
> Source: [vrcorp/TelePhotos](https://github.com/vrcorp/TelePhotos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
