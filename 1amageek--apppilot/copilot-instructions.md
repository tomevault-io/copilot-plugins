## apppilot

> - `Sources/AppPilot/` hosts the library code: `Core` for shared models, `Driver` for accessibility and CGEvent adapters, `Pilot` for public actors, and `Support` for utilities.

# Repository Guidelines

## Project Structure & Module Organization
- `Sources/AppPilot/` hosts the library code: `Core` for shared models, `Driver` for accessibility and CGEvent adapters, `Pilot` for public actors, and `Support` for utilities.
- `Tests/AppPilotTests/` contains Swift Testing suites; mirror the source tree and co-locate fixtures with the feature under test.
- `TestApp/` provides the macOS harness exercised by integration tests; keep storyboard identifiers and accessibility titles stable.
- `test-results/` stores captured screenshots or logs that accompany investigations—purge large binaries before committing.

## Build, Test, and Development Commands
- `swift build` — compile with the Swift 6.1 toolchain defined in `Package.swift`.
- `swift test` — execute all test targets; must pass before you push.
- `swift test --filter "Integration"` — run only window and AXUI integration suites while iterating on accessibility flows.
- `xcodebuild -project TestApp/TestApp.xcodeproj -scheme TestApp` — validate UI harness changes outside the SPM context.

## Coding Style & Naming Conventions
- Use 4-space indentation, Swift API Design Guideline casing (types PascalCase, members camelCase), and prefer expressive enum namespaces over free functions.
- Keep concurrency explicit: extend existing `actor` types or annotate with `@MainActor` when touching UI; avoid introducing custom queues.
- Limit `public` surface to the `Pilot` module and document reusable APIs with concise `///` summaries.

## Testing Guidelines
- Adopt behavior-driven test names (`testClickingButtonUpdatesAXTree`) and suffix suites with `.unit`, `.integration`, or `.performance` to aid filtering.
- Reset any apps or windows you spawn so repeated `swift test` runs remain deterministic; record flaky cases in `test-results/` for follow-up.
- When adding new fixtures, update both the corresponding `Tests/...` helper and `TestApp` so accessibility identifiers stay in sync.

## Commit & Pull Request Guidelines
- Follow the existing imperative summary style (`Move screenshot save location to /tmp`); keep the first line under ~72 characters.
- In commit bodies and PR descriptions, list motivation, key changes, and validation commands; add `Refs:` links for issues or docs as needed.
- PRs should include reproduction steps, screenshots for UI adjustments, and a note confirming Accessibility permissions were granted during testing.

## Accessibility & Environment Setup
- Enable Accessibility access for the test runner (System Settings → Privacy & Security → Accessibility) before running integration suites or the `TestApp` harness.
- Target macOS 15.0+ and Xcode 16.x to match the manifest; call out deviations so reviewers can replicate your environment.

---
> Source: [1amageek/AppPilot](https://github.com/1amageek/AppPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
