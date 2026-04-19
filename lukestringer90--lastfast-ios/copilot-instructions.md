## lastfast-ios

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

This is a native Xcode project — no Makefile, Fastlane, or build scripts. Use `xcodebuild` or open in Xcode 15+.

```bash
# Build
xcodebuild -scheme LastFast -destination 'platform=iOS Simulator,name=iPhone 16' build

# Run all unit tests
xcodebuild -scheme LastFast -destination 'platform=iOS Simulator,name=iPhone 16' test

# Run a single test class
xcodebuild -scheme LastFast -destination 'platform=iOS Simulator,name=iPhone 16' -only-testing:LastFastUnitTests/FastingSessionTests test
```

UI tests accept launch arguments `--clear-data` and `--ui-test-goal` to control test state.

There is no linter configured — Xcode warnings are the only static analysis.

## Architecture

SwiftUI + SwiftData app targeting iOS 17.0+. No UIKit. No third-party UI frameworks.

**Targets:**
- `LastFast` — main app
- `LastFastWidget` — WidgetKit extension (home/lock screen widgets + Live Activities)
- `LastFastUnitTests` / `LastFastUITests` — test targets

**Layers:**
```
SwiftUI Views
  └── Services (AnalyticsManager, NotificationManager, LiveActivityManager)
        └── SwiftData Models (FastingSession) + CloudKit sync
```

- **Persistence**: SwiftData `@Model` with CloudKit sync via `iCloud.dev.stringer.lastfast`
- **Widget data sharing**: `UserDefaults` over App Group `group.dev.stringer.lastfast.shared`
- **Analytics**: Firebase Analytics/Crashlytics, initially disabled until ATT permission granted
- **Siri**: 3 App Intents — Start, Stop, Status fasting

**Key source locations:**
- `LastFast/Main/Views/` — primary fasting timer UI
- `LastFast/History/` — history list, graph, and chart
- `LastFast/Services/` — all service classes
- `LastFast/Shared/` — design system, models, utilities
- `Shared/` — `FastingSession` model shared between app and widget targets

## Feature Flags

Defined as constants in the codebase:
- `useGraphHistoryView` — `true` = graph view, `false` = list view in history
- `liveActivityEnabled` — Dynamic Island Live Activity (currently `false`)

## Data Model

`FastingSession` (SwiftData `@Model`) is the central model — tracks start/end time and goal. Default goal is `defaultFastingGoalMinutes = 960` (16 hours). The widget reads session data via `DataSnapshotManager` which serialises to the shared App Group.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lukestringer90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
