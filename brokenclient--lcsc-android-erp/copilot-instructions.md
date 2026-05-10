## lcsc-android-erp

> This repository is a single-module Android app in `app/`. Kotlin source lives under `app/src/main/java/com/example/lcsc_android_erp`, organized by layer and feature:

# Repository Guidelines

## Project Structure & Module Organization

This repository is a single-module Android app in `app/`. Kotlin source lives under `app/src/main/java/com/example/lcsc_android_erp`, organized by layer and feature:

- `feature/`: Compose screens and view models for `home`, `inbound`, `inventory`, `search`, and `settings`
- `core/`: app container, Room database, DataStore, network, and shared UI helpers
- `data/`: repository implementations, backup/export, image persistence, and remote data sources
- `domain/`: repository interfaces and app models
- `ui/`: app shell and theme

Resources are in `app/src/main/res`. Room schemas are exported to `app/schemas`. Design and planning notes live in `docs/`. Crash logs are stored in `log/`.

## Build, Test, and Development Commands

Use the Gradle wrapper from the repo root:

- `./gradlew :app:compileDebugKotlin` — fast compile check for Kotlin/Compose changes
- `./gradlew :app:assembleDebug` — build a debug APK
- `./gradlew :app:testDebugUnitTest` — run local unit tests
- `./gradlew :app:connectedDebugAndroidTest` — run instrumentation/UI tests on a device or emulator

For local Android Studio work, open the root project and run the `app` configuration.

## Coding Style & Naming Conventions

- Follow Kotlin defaults: 4-space indentation, no tabs, concise functions, and expression-style code when readable
- Keep feature UI in `feature/<name>/`, business/data code in `data/` or `domain/`
- Name screens and dialogs with `...Screen`, `...Dialog`, `...Route`
- Name view models `...ViewModel`, UI state models `...UiState`, and shared Compose cards `...Card`
- Reuse existing shared components such as `MaterialInboundDialog` and `core/ui/MaterialListCard` before adding new UI variants

## Testing Guidelines

Current test roots:

- `app/src/test/` for local JVM tests
- `app/src/androidTest/` for instrumentation tests

Prefer adding unit tests for parsers, sort rules, BOM matching, and repository logic. Name tests after the behavior being verified, for example `matches resistor BOM rows by value`.

## Commit & Pull Request Guidelines

Git history is not available in this workspace, so use a simple imperative style:

- `inventory: fix location detail image loading`
- `search: refresh BOM match after direct inbound`

PRs should include:

- a short summary of user-visible changes
- affected screens or flows
- screenshots for Compose UI changes
- verification notes, such as `./gradlew :app:compileDebugKotlin`

## Architecture Notes

This app uses Jetpack Compose + Room + DataStore + Retrofit/Jsoup. Preserve the existing flow: UI -> ViewModel -> Repository -> Room/network. Keep persisted schema changes compatible and update `app/schemas` when Room entities change.

---
> Source: [BrokenClient/LCSC_android_erp](https://github.com/BrokenClient/LCSC_android_erp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
