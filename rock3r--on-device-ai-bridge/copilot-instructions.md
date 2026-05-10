## on-device-ai-bridge

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

An IntelliJ platform plugin for Apple Intelligence integration. Built from the [JetBrains IntelliJ Platform Plugin Template](https://github.com/JetBrains/intellij-platform-plugin-template).

## Build Commands

All commands use the Gradle wrapper (`./gradlew`). JDK 21 is required (configured via Gradle toolchain).

- **Build:** `./gradlew build`
- **Run plugin in sandbox IDE:** `./gradlew runIde`
- **Run tests:** `./gradlew test`
- **Run a single test:** `./gradlew test --tests "org.jetbrains.plugins.apple.ai.AppleAiPluginTest.testMethodName"`
- **Verify plugin compatibility:** `./gradlew verifyPlugin`
- **Build plugin distribution:** `./gradlew buildPlugin` (output in `build/distributions/`)

## Static analysis

You are NOT allowed to change static analysis settings without explicit user consent.

Current setup:
- **Detekt** via `io.gitlab.arturbosch.detekt`, configured in `detekt.yml` and `build.gradle.kts`.
- **ktfmt** (Kotlin formatter) configured in `build.gradle.kts`.
- **Kover** coverage with XML report on `check`.

IMPORTANT: When applicable, prefer using intellij-index MCP tools for code navigation, refactoring, and to access the
IntelliJ IDEA inspections, which can provide extra insights and spot more bugs.

## Architecture

This is a Kotlin-based IntelliJ Platform plugin using the [IntelliJ Platform Gradle Plugin 2.x](https://plugins.jetbrains.com/docs/intellij/tools-intellij-platform-gradle-plugin.html).

### Key Configuration

- **`gradle.properties`** — Central config: plugin ID/version, target platform version, `sinceBuild`, plugin dependencies. Modify this file (not `build.gradle.kts`) when changing versions or adding platform plugin dependencies.
- **`gradle/libs.versions.toml`** — Gradle version catalog for build tool dependencies.
- **`src/main/resources/META-INF/plugin.xml`** — Plugin descriptor. Register all extensions, actions, services, and listeners here.
- **`src/main/resources/messages/AppleAiBundle.properties`** — Localized strings accessed via `AppleAiBundle.kt`.

### Source Layout

- `src/main/kotlin/` — Plugin source code (Kotlin), main package `org.jetbrains.plugins.apple.ai`:
  - Services: `AppleAiService`, `AppleAiSettings`
  - Startup: `AppleAiStartupListener`
  - REST: `AppleAiRestService` (OpenAI-compatible HTTP handler)
  - UI/Actions: `AppleAiConfigurable`, `AppleAiToggleServerAction`, `AppleAiToggleDebugLogsAction`
  - Runtime: `AppleAiCustomPortServerManager`, `SwiftHelperProcess`, `OpenAiModels`
- `src/main/resources/` — Plugin resources:
  - `META-INF/plugin.xml` for extensions/actions/listeners
  - `messages/AppleAiBundle.properties` for localized strings
  - `META-INF/pluginIcon.svg` and `pluginIcon_dark.svg`
  - `swift-sources/` for helper process sources
- `src/test/kotlin/` — Tests (JUnit 4 with IntelliJ test framework fixtures)
- `src/test/testData/` — Test fixture data files

### Build Features

- **README.md plugin description:** The build extracts text between `<!-- Plugin description -->` and `<!-- Plugin description end -->` markers in README.md for the plugin marketplace listing.
- **CHANGELOG.md:** Managed by the [Gradle Changelog Plugin](https://github.com/JetBrains/gradle-changelog-plugin). Change notes are pulled from here during builds.
- **Gradle Configuration Cache** and **Build Cache** are enabled.
- **Kover** is configured for code coverage (XML report on `check`).

### CI/CD

- `.github/workflows/build.yml` — Build, test, verify on push/PR
- `.github/workflows/release.yml` — Publish to JetBrains Marketplace on release
- Plugin signing and publishing use environment variables: `CERTIFICATE_CHAIN`, `PRIVATE_KEY`, `PRIVATE_KEY_PASSWORD`, `PUBLISH_TOKEN`

---
> Source: [rock3r/on-device-ai-bridge](https://github.com/rock3r/on-device-ai-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
