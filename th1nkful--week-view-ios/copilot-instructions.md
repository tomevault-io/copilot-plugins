## week-view-ios

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

This is a pure Xcode project (no SPM dependencies, no CocoaPods). Build and run using:

```bash
xcodebuild -project WeekView.xcodeproj -scheme WeekView -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max' build
```

There are no test targets configured. No linter is set up.

## Architecture

MVVM pattern using SwiftUI with `@StateObject` ViewModels as `@MainActor ObservableObject` classes.

- **Models** (`WeekView/Models/`) — Thin wrappers around EventKit/WeatherKit types (`EKEvent`, `EKReminder`, `CurrentWeather`) that provide formatted display strings
- **ViewModels** (`WeekView/ViewModels/`) — `CalendarViewModel` manages EventKit permissions and event/reminder fetching; `WeatherViewModel` manages CoreLocation and WeatherKit (currently disabled)
- **Views** (`WeekView/Views/`) — Composable SwiftUI views

## Key Implementation Details

**ContentView.swift** is the root view and contains significant inline logic including `InfiniteDayScrollView` and `DaySection` as nested structs. This is the largest file (~383 lines) and handles scroll state management, lazy event loading, and date selection.

**Week navigation** uses a horizontal `ScrollView` with a range of -52 to +52 weeks from current. Weeks start on Monday (`calendar.firstWeekday = 2`). Adjacent weeks load on-demand with 300ms debouncing.

**EventKit permissions** use iOS 17.0+ APIs (`requestFullAccessToEvents()` / `requestFullAccessToReminders()`). Events load per-date via predicates on `EKEventStore`.

**Deep linking**: Events open via `calshow://`, reminders via `x-apple-reminderkit://`.

**Deployment target**: iOS 17.0. Swift 5. No external dependencies — only system frameworks (SwiftUI, EventKit, WeatherKit, CoreLocation).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/th1nkful) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
