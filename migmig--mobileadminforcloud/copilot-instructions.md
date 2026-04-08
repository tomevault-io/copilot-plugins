## mobileadminforcloud

> This file is for coding agents working in `MobileAdminForCloud`.

# AGENTS.md

This file is for coding agents working in `MobileAdminForCloud`.

## Project Summary

- Cross-platform iOS/macOS admin dashboard built with SwiftUI.
- Swift + SwiftData, JWT auth, biometric auth on iOS.
- No external dependencies; prefer Apple frameworks already in use.
- MVVM-ish architecture with `ViewModel` as the main observable facade and newer logic under `MobileAdmin/model/services/`.

## Source of Truth

- Follow this file first for agent workflow in this repo.
- Also follow `CLAUDE.md` at the repository root for architecture and folder conventions.
- No `.cursor/rules/`, `.cursorrules`, `.github/copilot-instructions.md`, or in-repo PRD/Spec/Tasks docs tree exist today.

## Important Paths

- App entry: `MobileAdmin/MobileAdminApp.swift`
- Scenes: `MobileAdmin/Scense/`
- Shared views: `MobileAdmin/views/common/`
- Platform views: `MobileAdmin/views/ios/`, `MobileAdmin/views/macos/`
- Models/services: `MobileAdmin/model/`, especially `Cloud/`, `DevTools/`, `services/`
- Utilities/design system: `MobileAdmin/Util/`
- Tests: `MobileAdminTests/`, `MobileAdminUITests/`

## Build, Test, and Analysis Commands

This is a native Xcode project. There is no SwiftPM package, CocoaPods, or Carthage.

### Full build

```bash
xcodebuild -project MobileAdmin.xcodeproj -scheme MobileAdmin -destination 'platform=iOS Simulator,name=iPhone 16'
```

### Full unit test run

```bash
xcodebuild test -project MobileAdmin.xcodeproj -scheme MobileAdmin -testPlan MobileAdmin -destination 'platform=iOS Simulator,name=iPhone 16'
```

### Run a single Swift Testing suite

```bash
xcodebuild test -project MobileAdmin.xcodeproj -scheme MobileAdmin -testPlan MobileAdmin -destination 'platform=iOS Simulator,name=iPhone 16' -only-testing:'MobileAdminTests/EnvironmentConfigTests'
```

### Run a single Swift Testing test method

Quote the identifier and keep the `()` suffix.

```bash
xcodebuild test -project MobileAdmin.xcodeproj -scheme MobileAdmin -testPlan MobileAdmin -destination 'platform=iOS Simulator,name=iPhone 16' -only-testing:"MobileAdminTests/EnvironmentConfigTests/current_canBeSetToDevelopment()"
```

### Run XCTest-based UI tests

```bash
xcodebuild test -project MobileAdmin.xcodeproj -scheme MobileAdmin -destination 'platform=iOS Simulator,name=iPhone 16' -only-testing:'MobileAdminUITests/MobileAdminUITests'
```

### List available destinations

```bash
xcodebuild -project MobileAdmin.xcodeproj -scheme MobileAdmin -showdestinations
```

### Static analysis

The scheme has `buildForAnalyzing="YES"`, so `analyze` is valid.

```bash
xcodebuild analyze -project MobileAdmin.xcodeproj -scheme MobileAdmin -destination 'platform=iOS Simulator,name=iPhone 16'
```

## Lint / Format Status

- No `swiftlint` config found.
- No `swiftformat` config found.
- Do not invent a formatter regime for small edits; match the surrounding file unless the task is explicitly cleanup.

## Testing Framework Notes

- Unit tests use Apple’s `Testing` framework with `@Test` and `#expect`.
- UI tests use XCTest.
- `MobileAdmin.xctestplan` currently includes the `MobileAdminTests` target.
- When tests mutate shared static state, the repo already uses `@Suite(.serialized)` where needed.

## Code Style Guidelines

### Imports

- Keep imports minimal and Apple-framework-only.
- Common patterns: views use `SwiftUI`; services/models use `Foundation`; logging uses `Logging`; persistence uses `SwiftData`.
- Guard platform-specific imports with `#if os(iOS)` / `#if os(macOS)`.

### Formatting

- Newer files use standard Swift spacing, 4-space indentation, blank lines between sections, and `// MARK: -`.
- Older files use compact legacy formatting; do not reformat unrelated code. Keep the diff narrow.

### Naming

- Types are `UpperCamelCase`; variables, functions, and properties are `lowerCamelCase`.
- Test names are descriptive and behavior-oriented, often snake-like with underscores.
- Respect existing file suffixes and naming patterns like `*ViewForIOS.swift`, `*ViewIOS.swift`, `*ForMac.swift`, `*Sidebar.swift`, `*Detail*.swift`, and `Source*.swift`.
- Preserve repo quirks unless the task is explicitly to rename/fix them: `Scense/`, `SlidebarViewForMac`, and existing typoed identifiers.

### Architecture and Placement

- Shared views go in `MobileAdmin/views/common/`; iOS-only in `views/ios/`; macOS-only in `views/macos/`.
- API/data models belong in `model/Cloud/` or `model/DevTools/`; newer networking/domain logic belongs in `model/services/`.
- Keep `ViewModel.swift` as the central observable facade unless an existing service already owns the behavior.

### SwiftUI Patterns

- Use `@StateObject` when the view owns an observable object and `@EnvironmentObject` for injected shared objects.
- Many existing views also use `@ObservedObject var viewModel`; follow the local pattern instead of forcing migration.
- Use `@Binding` for parent-owned mutable state, and `@AppStorage` / `@Query` where the feature already relies on them.
- Cross-platform branching should be compile-time via `#if os(iOS)` / `#elseif os(macOS)`.

### Async and Loading

- Prefer `async/await`.
- In views, load data with `.task`, `.refreshable`, or explicit `Task {}` for user actions.
- Use local `isLoading` state when that matches nearby code.
- Prefer existing helpers like `.loadingTask(isLoading:action:)` in newer design-system areas.

### Types and Models

- API models should conform to `Codable`.
- Add `Identifiable` and `Hashable` when SwiftUI lists or selections require them.
- Existing models often provide an empty `init()` plus fuller initializers; match nearby style.
- `EnvironmentModel` is the only SwiftData `@Model` called out in repo docs.

### Error Handling

- Prefer typed errors already present in the repo, especially `NetworkError`.
- In infrastructure code, throw meaningful typed errors.
- In service code, match the existing pattern: catch, log, and return nil/empty fallback where that service already behaves that way.
- Do not add empty `catch` blocks or placeholder suppression.

### Comments and Language

- Comments are commonly written in Korean.
- English comments are acceptable when needed, but do not rewrite existing Korean comments unless requested.
- Prefer clear section comments over noisy inline narration.

## Security and Sensitive Data

- Do not expose or log the Base64-encoded `adminCI` credential in `MobileAdmin/config/Info.plist`.
- Do not commit secrets or derived credentials.
- JWT tokens are memory-held; do not add persistence for them unless explicitly required.
- ATS currently allows arbitrary loads for local/dev support; do not “tighten” this casually.

## Change Strategy for Agents

- Inspect nearby files first and match local conventions.
- Prefer small, surgical diffs; avoid unrelated renames or formatting sweeps.
- If a newer service-layer pattern exists nearby, extend it instead of adding parallel logic in the view.
- New API endpoint: add request/response models in the right folder, add the call in `ViewModel.swift` or the relevant service, and add `@Published` state only if the UI needs observation.
- New view: wire it into the appropriate iOS or macOS navigation entry points.

## Pre-Completion Checklist

Before claiming work is done:

- Run the most relevant test command you can for the changed area.
- Prefer single-test execution first, then broader coverage.
- If you changed app code, run at least a build or analyze command in a real Xcode environment.
- Re-read the diff for accidental formatting churn and confirm you did not expose credentials or alter unrelated platform behavior.

## Practical Reminder

This repo is transitional: some files follow newer style and some retain older compact conventions. Match the file you are in, keep the diff tight, and improve structure only when the task actually calls for it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/migmig)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/migmig)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
