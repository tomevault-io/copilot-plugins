## kototoro

> `app/` contains the Android application. Main Kotlin sources live under `app/src/main/kotlin/org/skepsun/kototoro` and are organized by feature with `data`, `domain`, and `ui` layers. Android resources, menus, themes, and preference XML files live in `app/src/main/res`. Shared parser contracts are in `parser-api/`. Unit tests belong in `app/src/test/kotlin`, instrumented tests in `app/src/androidTest/kotlin`, and Room schemas in `app/schemas/`. Documentation lives in `docs/` and is built with VitePress from the root `package.json`.

# Repository Guidelines

## Project Structure & Module Organization
`app/` contains the Android application. Main Kotlin sources live under `app/src/main/kotlin/org/skepsun/kototoro` and are organized by feature with `data`, `domain`, and `ui` layers. Android resources, menus, themes, and preference XML files live in `app/src/main/res`. Shared parser contracts are in `parser-api/`. Unit tests belong in `app/src/test/kotlin`, instrumented tests in `app/src/androidTest/kotlin`, and Room schemas in `app/schemas/`. Documentation lives in `docs/` and is built with VitePress from the root `package.json`.

## Build, Test, and Development Commands
Use the bundled wrappers and keep commands scoped:

- `./gradlew :app:assembleDebug` builds a debug APK.
- `./gradlew :app:compileDebugKotlin --no-daemon` is the fastest compile-only validation for Kotlin changes.
- `./gradlew :app:testDebugUnitTest --no-daemon` runs JVM unit tests.
- `./gradlew :app:connectedDebugAndroidTest` runs instrumented tests on a device or emulator.
- `npm ci && npm run docs:dev` starts the local docs site.
- `npm run docs:build` builds the static docs output.

## Coding Style & Naming Conventions
Follow `.editorconfig`: UTF-8, LF, 4-space indentation, and 120-character line width. Kotlin uses the official style with trailing commas enabled. Name classes and test classes in `PascalCase`, methods and properties in `camelCase`, and Android resources in lowercase underscore style such as `pref_appearance.xml` or `mode_chapters.xml`. Prefer extending existing feature modules instead of creating parallel implementations. Keep comments short and in the same language already used nearby.

## Testing Guidelines
The project uses JUnit5, Kotest, MockK, and MockWebServer for unit tests, plus AndroidX Test, Hilt, and Room testing. Match test file names to the target type, typically `*Test.kt`, `*IntegrationTest.kt`, or `*PropertyTest.kt`. Changes to parser flows, networking, database code, downloads, or reader behavior should include focused coverage or at least a compile/test pass before review.

## Commit & Pull Request Guidelines
Recent history mixes short imperative subjects with Conventional Commits, and the latter is preferred, for example `feat: ...`, `fix(scope): ...`, `docs: ...`, or `chore: ...`. Keep each commit narrowly scoped. Pull requests should explain motivation, affected areas, validation commands, and risks. Include screenshots for UI, reader, download, or documentation changes, and link the related issue when one exists.

## Security & Contributor Notes
Do not commit `local.properties`, signing files, secrets, caches, or generated artifacts. Translation content is managed through Weblate, so avoid bulk manual string rewrites unless fixing a clear defect. Keep `README.md` product-focused; put deeper engineering notes in `docs/`. When touching download, translation, super-resolution, or local storage flows, verify both UI entry points and the background task pipeline.

---
> Source: [Kototoro-app/Kototoro](https://github.com/Kototoro-app/Kototoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
