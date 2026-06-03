## swift-explorer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

Swift Explorer is a macOS desktop app that lets users paste Swift code and see the resulting LLVM IR and assembly output, similar to Compiler Explorer. It uses `swiftc -emit-ir` and `swiftc -S` under the hood.

## Commands

**Build & Test (Xcode 26.2.0 required — see `.xcode-version`):**
```bash
# Run all tests
xcodebuild clean test -project SwiftExplorer.xcodeproj -scheme SwiftExplorer -skipPackagePluginValidation

# Run tests for a specific SPM package
xcodebuild test -project SwiftExplorer.xcodeproj -scheme SwiftExplorer -only-testing:SwiftExplorerTests/HomeViewModelTests
```

**Lint:**
```bash
swiftlint
```

**Localization strings (requires mise):**
```bash
mise run swiftgen
```

## Architecture

The project uses MVVM with a local Swift Package Manager monorepo under `Dependencies/`. The main Xcode project depends on these packages:

- **Lowlevel** — Core logic. `Llvm.swift` runs `swiftc -emit-ir`, `Assembly.swift` runs `swiftc -S`. `OptimizationLevel` enum maps to compiler flags (`-O`, `-Onone`, `-Osize`, `-Ounchecked`).
- **Analytics** — Firebase Analytics + Crashlytics wrappers. All event names live in `Events.swift` as nested enums (`AnalyticsEvents.Home`, `CrashlyticsEvents.Home`).
- **DesignSystem** — `CustomCodeEditorView` (wraps a CodeEditor fork with Swift/Bash syntax highlighting), `ButtonSelect`, color/font theme extensions.
- **Common** — `AppLinks`, `Bundle+AppVersion`.
- **CommonTest** — `TestBase`, SwiftUI test helpers shared across package tests.

**Main app layer** (`SwiftExplorer/`):
- `HomeView` — Three code editors (Swift input, LLVM IR output, Assembly output) plus an optimization-level picker and an Explore button.
- `HomeViewModel` — `@Observable` class, holds the three code strings and optimization level. `generate()` calls `Lowlevel` to produce IR and assembly. Accepts `AnalyticsTracking` and `CrashlyticsTracking` via protocol injection.
- `AboutView` — Static about screen.

## Testing Patterns

- ViewModels are tested in `SwiftExplorerTests/` using stub implementations of the Analytics/Crashlytics protocols (e.g., `AnalyticsTrackingStub`).
- Snapshot tests live in `SwiftExplorerTests/__Snapshots__/` — update them intentionally when UI changes.
- Each SPM package has its own `Tests/` target tested through the main scheme.

## Code Conventions

- SwiftLint is enforced; key custom rule: all classes must be `final`.
- Localizable strings are code-generated via SwiftGen into `L10n.swift` — add new strings to the `.strings` file and regenerate, never hardcode UI text.
- Firebase is initialized in `SwiftExplorerApp.swift` on launch; the Analytics module is the only place Firebase is imported.

---
> Source: [heroesofcode/swift-explorer](https://github.com/heroesofcode/swift-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
