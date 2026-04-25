## threedailygoals

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build main app
xcodebuild -project "Three Daily Goals.xcodeproj" -scheme "Three Daily Goals" -configuration Debug build

# Run unit tests (always run before committing)
xcodebuild test -project "Three Daily Goals.xcodeproj" -scheme "Three Daily Goals" \
  -destination "platform=iOS Simulator,name=iPhone 15,OS=latest" \
  -only-testing:Three_Daily_GoalsTests

# Run a single test
xcodebuild test -project "Three Daily Goals.xcodeproj" -scheme "Three Daily Goals" \
  -destination "platform=iOS Simulator,name=iPhone 15,OS=latest" \
  -only-testing:Three_Daily_GoalsTests/TestClassName/testMethodName

# Swift package tests
cd tdgCore && swift test && swift build
```

**Never run** `Three Daily GoalsUITests` or `run_share_extension_tests.sh` unless explicitly requested.

After every change: ensure all tests in `Three Daily GoalsTests` compile and pass, and new features have tests. Remove unnecessary comments.

## Architecture

### Multi-Module Swift Package (`tdgCore`)

Dependency flow (each depends on all above it):
```
tdgCoreWidget  →  base utilities, helpers, SF Symbol constants, TimeProvider
tdgCoreMain    →  TaskItem/Comment/Attachment models, SwiftData schemas/migrations
tdgCoreShare   →  share extension logic (ShareFlow.resolve())
tdgCoreTest    →  test utilities and mock objects
```

The main app target adds controllers, views, and the Compass Check workflow on top of `tdgCoreMain`.

### App Initialization

`Three_Daily_GoalsApp.swift` → `setupApp()` (AppSetup.swift) creates all managers and injects them into the SwiftUI environment:

- **DataManager** — all data operations (CRUD, queries, import/export, undo/redo)
- **UIStateManager** — UI state (selected task, dialogs, navigation, search)
- **CompassCheckManager** — daily review workflow
- **CloudPreferences** — user settings synced via CloudKit key-value store
- **CalendarManager** — EventKit integration

Root view is `MainView.swift` → `RegularMainView` (iPad/Mac) or `CompactMainView` (iPhone).

### Domain Models (SwiftData)

Current schema: `SchemaV3_6` (aliased as `SchemaLatest`). All schema versions in `tdgCore/Sources/tdgCoreMain/Storage/ModelVersions/`, migrations in `Migrations.swift`.

**TaskItem** has 5 states: `open`, `priority`, `pendingResponse`, `closed`, `dead` (graveyard). State transitions create `Comment` audit entries. Use transition methods (`closeTask()`, `makePriority()`, `graveyard()`) — never assign `_state` directly. Tags stored as comma-separated string in `allTagsString`.

When modifying models: create a new schema version file, add migration stages to `TDGMigrationPlan`, update `SchemaLatest`.

### Compass Check System

A customizable 10-step daily review workflow. Steps conform to `CompassCheckStep` protocol and live in `Domain/CompassCheckSteps/`. Steps can be enabled/disabled via preferences. `CompassCheckManager` handles state and step execution.

## Key Conventions

### SF Symbols
**All SF Symbol strings must be declared as constants in `tdgCore/Sources/tdgCoreWidget/Helpers/IconsRelated.swift`** before use. Never use hardcoded strings like `"star.fill"` inline. Constants use `img` prefix (e.g., `imgPriority`, `imgPlus`).

### Time Abstraction
Never use `Date()` directly. Always use `timeProvider.now` through the `TimeProvider` protocol (`RealTimeProvider` in production, `MockTimeProvider` in tests).

### Test Mode
App detects `enable-testing` in `CommandLine.arguments` to use an in-memory `ModelContainer` with `createDefaultTestData()` instead of CloudKit-synced storage.

### TDD Requirement
Write a failing test first for every bug fix or new feature. The workflow: failing test → implementation → verify passing. Never fix a bug without a failing test first.

## Critical Rules

- **Cross-platform**: All code must run on both iOS and macOS. Only use UIKit/AppKit where both platforms are handled.
- **CloudKit**: Do **not** use `@Attribute(.unique)` — it's incompatible with CloudKit sync.
- **Concurrency**: Use `Task { @MainActor in }` instead of `DispatchQueue.main.async`. All managers are `@MainActor`. Swift 6 strict concurrency is enabled.
- **Custom colors**: When used with `.foregroundStyle()`, wrap in `Color` type: `.foregroundStyle(Color.open)` not `.foregroundStyle(.open)`.
- **Only commit when explicitly requested.**
- **Do not use git worktrees.** Work directly in the main repository.

## SwiftUI Patterns

Prefer modern APIs:
- `.foregroundStyle()` over `.foregroundColor()`
- `.clipShape(.rect(cornerRadius:))` over `.cornerRadius()`
- `NavigationStack` + `NavigationLink(value:)` + `.navigationDestination(for:)` over `NavigationView`
- `@Observable` over `ObservableObject`; `@Bindable` over `@ObservedObject`
- 2-parameter `.onChange(of:)` (1-parameter variant is deprecated)
- Split views into separate `struct` types rather than computed `var` properties (critical for `@Observable` invalidation performance)
- `Button` over `.onTapGesture()` for interactive elements
- Dynamic Type fonts (`.body`, `.headline`) over fixed `system(size:)` fonts
- `Task { @MainActor in }` over `DispatchQueue.main.async`

## PR Conventions

Title format: `[Component] Description` — e.g., `[CompassCheck] Fix step navigation bug`, `[Storage] Add migration for SchemaV3_7`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vithanco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
