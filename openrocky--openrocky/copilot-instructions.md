## openrocky

> OpenRocky is split into three active areas:

# Repository Guidelines

## Project Structure & Module Organization
OpenRocky is split into three active areas:
- `OpenRocky/`: main iOS app (`OpenRocky.xcodeproj`), app source in `OpenRocky/OpenRocky/`, unit tests in `OpenRocky/OpenRockyTests/`, UI tests in `OpenRocky/OpenRockyUITests/`.
- `Packages/`: local Swift packages used by the app:
  - `LanguageModelChatUI`, `MarkdownViewLocal`, `SwiftOpenAI`
  - `OpenRockyIOSSystem` and `OpenRockyPython` (runtime/system integrations)
- `website/`: Docusaurus docs site (`website/docs`, `website/src`, `website/static`).

Build artifacts are generated under `build/`, `OpenRocky/build/`, and `website/build/`; avoid committing generated output.

## Build, Test, and Development Commands
- App build (repo root):
  ```bash
  xcodebuild build -scheme OpenRocky -project OpenRocky/OpenRocky.xcodeproj -destination 'generic/platform=iOS'
  ```
  Verifies iOS app compiles for device.
- App tests:
  ```bash
  xcodebuild test -scheme OpenRocky -project OpenRocky/OpenRocky.xcodeproj -destination 'platform=iOS Simulator,name=iPhone 16'
  ```
- Swift package tests (example):
  ```bash
  swift test --package-path Packages/LanguageModelChatUI
  ```
- Python runtime setup (required by OpenRockyPython):
  ```bash
  scripts/setup_python.sh
  ```
- TestFlight pipeline:
  ```bash
  scripts/testflight.sh all
  ```
- Website (inside `website/`): `npm run start`, `npm run build`, `npm run typecheck`.

## Coding Style & Naming Conventions
Use Swift 6 conventions: 4-space indentation, `UpperCamelCase` for types, `lowerCamelCase` for methods/properties, and clear enum case names (`OpenRockyProviderKind.openAI` style). Keep feature views grouped under `OpenRocky/OpenRocky/Features/<FeatureName>/`.

`SwiftOpenAI` includes formatter rules at `Packages/SwiftOpenAI/rules.swiftformat`:
```bash
swiftformat --config Packages/SwiftOpenAI/rules.swiftformat Packages/SwiftOpenAI
```

## Testing Guidelines
Prefer small, deterministic unit tests in `OpenRockyTests` (using Swift `Testing` + `#expect`). UI flow checks belong in `OpenRockyUITests` (`XCTest`). Name tests by behavior, e.g. `providerInventoryIncludesConfiguredBackends`.

Run impacted package tests plus app tests before opening a PR.

## Commit & Pull Request Guidelines
Recent history favors concise Conventional Commits: `feat: ...`, `fix: ...`, `chore: ...` (occasional `Revert ...`). Follow that format in imperative mood.

PRs should include:
- What changed and why (scope + risk)
- Linked issue/task (if any)
- Screenshots/video for UI updates (`OpenRocky/` or `website/`)
- Exact verification commands you ran

---
> Source: [openrocky/openrocky](https://github.com/openrocky/openrocky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
