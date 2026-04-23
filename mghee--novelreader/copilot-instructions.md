## novelreader

> Purpose: onboard coding agents to this repository quickly with minimal, universally useful context.

# CLAUDE.md

Purpose: onboard coding agents to this repository quickly with minimal, universally useful context.

## WHY
NovelReader is an Android app for reading web novels, tracking reading progress, and syncing with a companion web tracker.
Primary goals:
- Stable reading experience (reader, chapters, sources, library)
- Reliable data and sync behavior
- Maintainable modular Android codebase

## WHAT (stack and repo map)
Tech stack:
- Kotlin, Gradle Kotlin DSL, Android multi-module project
- Jetpack Compose + Material 3 UI
- Room for local database
- Retrofit + OkHttp for networking
- Hilt for dependency injection
- WorkManager for background work

High-level modules:
- :app -> Android application entrypoint and module wiring
- :features:* -> user-facing feature modules (reader, chaptersList, sourceExplorer, catalogExplorer, settings, libraryExplorer, webview, etc.)
- :data -> data layer and repositories
- :networking -> network abstractions and clients
- :core and :coreui -> shared domain/ui building blocks
- :navigation -> app navigation graph and routing
- :scraper -> source scraping/parsing logic
- :strings -> string resources support module
- :tooling:* -> cross-cutting tooling modules (local database, text-to-speech, backup, translators, epub, workers, algorithms)

Authoritative structure and dependencies:
- settings.gradle.kts
- gradle/libs.versions.toml
- app/build.gradle.kts

## HOW (work effectively)
General workflow:
1. Understand the task and identify the owning module(s).
2. Read nearby code first and follow existing patterns.
3. Make the smallest safe change that solves the request.
4. Verify with targeted Gradle tasks before finishing.

Build and verification commands (Windows PowerShell):
- Build app debug APK: .\gradlew :app:assembleDebug
- Run unit tests: .\gradlew testDebugUnitTest
- Run lint: .\gradlew lint
- Run instrumentation tests (device/emulator required): .\gradlew connectedDebugAndroidTest

When changes are module-scoped, prefer module-scoped tasks first (example):
- .\gradlew :features:reader:assemble

## Progressive disclosure (read only when relevant)
Do not preload everything. Pull focused context based on task:
- Product and setup overview: README.md
- Full module graph: settings.gradle.kts
- App wiring and app-level dependencies: app/build.gradle.kts
- Shared versions and plugins: gradle/libs.versions.toml
- Build conventions plugins: build-logic/convention/src
- Feature implementation details: features/<feature-name>/src
- Data and persistence behavior: data/src and tooling/local_database
- Source parsing behavior: scraper/src and tooling/local_source

## Guardrails
- Prefer deterministic tools (Gradle, lint, tests) over subjective style instructions.
- Do not add broad new architectural patterns unless the task requires them.
- Keep edits focused; avoid unrelated refactors.
- If behavior is unclear, verify by reading code paths instead of assuming.

## Notes
- Keep this file concise and universally applicable.
- If a new recurring workflow is needed, add a separate focused doc and reference it here instead of expanding this file heavily.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MGhee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
