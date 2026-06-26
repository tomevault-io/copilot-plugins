## movement-log

> This is a single-module Android app project.

# Repository Guidelines

## Project Structure & Module Organization

This is a single-module Android app project.

- `app/src/main/java/com/yamichi77/movement_log`: production Kotlin code (UI, navigation, data, service).
- `app/src/main/res`: Android resources (`values/`, `drawable/`, `mipmap/`, `xml/`).
- `app/src/test`: JVM unit tests (ViewModel, use case, gateway tests).
- `app/src/androidTest`: instrumented/UI tests for device or emulator runs.
- `docs/`: implementation notes and archived build/test logs.
- `scripts/`: PowerShell helpers for regression and connection E2E runs.

Keep new files in the matching package/layer (`ui`, `data`, `navigation`, `service`) and mirror test paths to source paths.

## Build, Test, and Development Commands

Use the Gradle wrapper from repository root.

- Run Gradle commands outside the sandbox environment (request approval when required).
- `.\gradlew.bat :app:assembleDebug`: build debug APK.
- `.\gradlew.bat :app:testDebugUnitTest`: run JVM unit tests.
- `.\gradlew.bat :app:connectedDebugAndroidTest`: run instrumented tests (requires online device/emulator).
- `.\gradlew.bat :app:lintDebug`: run Android lint checks.
- `.\scripts\run-regression-tests.ps1`: run project regression flow and write logs to `docs/build-logs/`.
- `.\scripts\run-connection-e2e.ps1`: run connection E2E scenario using `cert.txt`.

## Coding Style & Naming Conventions

- Language: Kotlin (JDK 11), Jetpack Compose, AGP 9.x.
- Indentation: 4 spaces; keep trailing commas where already used.
- Types: `UpperCamelCase`; functions/variables: `lowerCamelCase`; constants: `UPPER_SNAKE_CASE`.
- Prefer descriptive suffixes: `...ViewModel`, `...UiState`, `...Repository`, `...Screen`.
- Use Android Studio formatter and fix lint findings before review.

## Testing Guidelines

- Frameworks: JUnit4, kotlinx-coroutines-test, MockWebServer, AndroidX test/espresso/compose test.
- Test class naming: `*Test.kt` (unit) and `*UiTest.kt` (instrumented UI).
- Test method naming follows behavior style (e.g., `init_startsCollectingAndMapsUnknownCoordinate`).
- For features touching UI/data flow, add or update both unit tests and instrumented coverage when applicable.

## Commit & Pull Request Guidelines

Git history is short but uses concise subjects (examples: `add: ...`, `Initial commit`). Follow `type: short summary` style where possible (`add:`, `fix:`, `refactor:`), one logical change per commit.

PRs should include:

- purpose and scope,
- linked issue/task (if any),
- test evidence (commands run and result),
- screenshots/video for UI changes,
- notes for config changes (`local.properties`, `MAPS_API_KEY`, endpoint/cert settings).

## Security & Configuration Tips

- Do not commit secrets in `local.properties`, `cert.txt`, or endpoint credentials.
- Keep `sdk.dir` local-only and verify `MAPS_API_KEY` is injected via local configuration.

---
> Source: [yamichi77/movement-log](https://github.com/yamichi77/movement-log) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
