## aiyo

> Aiyo is a Kotlin Android app split into Gradle modules:

# Repository Guidelines

## Project Structure & Module Organization

Aiyo is a Kotlin Android app split into Gradle modules:

- `app/`: Android application entry point, navigation, Hilt bindings, manifests, and app resources.
- `ui/`: Jetpack Compose UI, theme, custom basics components, screens, and UI tests.
- `domain/`: Repository interfaces and domain models.
- `data/`: Room database, MMKV storage, remote API client, repository implementations, and data mappers.
- `metadata/`: F-Droid listing text, changelogs, icon, and screenshots.
- `scripts/git-hooks/`: project Git hook sources installed during Gradle pre-build.

Tests live under each module’s `src/test` and `src/androidTest` directories.

## Build, Test, and Development Commands

Use the Gradle wrapper via `sh` unless `gradlew` is executable locally:

- `sh ./gradlew :app:compileDebugKotlin`: compile-check the app and dependent modules.
- `sh ./gradlew :app:assembleDebug`: build a debug APK.
- `sh ./gradlew :app:testDebugUnitTest`: run app unit tests.
- `sh ./gradlew testDebugUnitTest`: run debug unit tests across modules.
- `sh ./gradlew ktlintCheck`: run Kotlin style checks.
- `sh ./gradlew ktlintFormat`: apply ktlint formatting where safe.

## Coding Style & Naming Conventions

Use Kotlin with 4-space indentation and existing Compose/MVVM patterns. Keep package names under `com.beradeep.aiyo`. Prefer clear names such as `ChatViewModel`, `SettingsUiState`, `MessageDao`, and `ModelRepositoryImpl`.

UI work must use components from `ui/src/main/java/com/beradeep/aiyo/ui/basics`. Do not use Material components directly unless explicitly requested. If a basics component is missing, call that out before substituting another UI toolkit.

Ktlint is configured in `build.gradle.kts`; wildcard imports, filename, enum entry case, and final newline rules are disabled.

## Testing Guidelines

Unit tests use JUnit and are placed in `src/test`. Android instrumentation tests use AndroidX test APIs and live in `src/androidTest`. Name tests after the behavior under test, for example `ChatViewModelTest` or `ConversationDaoTest`. Add focused tests for repository, mapper, database, and ViewModel logic when behavior changes.

## Commit & Pull Request Guidelines

Recent commits use short imperative or release-oriented messages, for example `Add configurable chat font sizes`, `fix lumo.properties`, and `changelog for release v1.0.8`. Keep commits focused and avoid unrelated generated or IDE files.

Pull requests should include a concise description, linked issue when applicable, test results, and screenshots or screen recordings for UI changes. Mention any migrations, new permissions, or configuration changes.

## Security & Configuration Tips

OpenRouter API keys are user-provided and should stay local. Do not commit secrets, local keystores, or machine-specific IDE files. Prefer repository abstractions over direct storage or network access from UI code.

---
> Source: [beradeep/aiyo](https://github.com/beradeep/aiyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
