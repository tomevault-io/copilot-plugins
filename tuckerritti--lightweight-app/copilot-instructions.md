## lightweight-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
# Build for simulator
xcodebuild -project light-weight.xcodeproj -scheme light-weight \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro' build

# No test targets exist yet
```

Requires Xcode 16.3+ and iOS 18.0+ deployment target. Single SPM dependency: AnthropicSwiftSDK (v0.14.0+).

## Simulator Testing

Each worktree gets its own simulator so agents can QA test in parallel.

**Setup:** Run `python3 ./scripts/setup-simulator.py` from the worktree root. This clones an iPhone 17 Pro simulator, builds the app, and installs it. The simulator UDID is saved to `.context/simulator-udid.txt`. Pass `--release` to build with the Release configuration (e.g. to test behavior without seed data or `#if DEBUG` code paths).

**Using MCP tools:** Read the UDID from `.context/simulator-udid.txt` and pass it as the `udid` parameter to all iOS simulator MCP tools (`ui_tap`, `ui_view`, `screenshot`, `ui_describe_all`, etc.).

**Rebuilding:** Run the script again after code changes — it reuses the existing simulator and rebuilds.

## Architecture

SwiftUI + SwiftData iOS app. AI-powered workout generator that uses Claude API (BYOK) and HealthKit for recovery-aware programming.

### Data Layer

Three SwiftData `@Model` classes: `Exercise`, `WorkoutLog`, `UserProfile`. Complex nested data is stored as JSON-encoded `Data` fields with computed property accessors (e.g. `WorkoutLog.entriesData` decoded to `[LogEntry]` via a computed `entries` property). This is the pattern used throughout to work around SwiftData's limitations with nested types.

Transient Codable structs (`Workout`, `WorkoutExercise`, `WorkoutSet`, `LogEntry`, `LogSet`) represent AI-generated and in-progress data that doesn't persist directly as models.

### AI Services (three tiers)

1. **ClaudeAPIService** — thin SDK wrapper. `send()` and `stream()` methods. Model hardcoded to `claude-sonnet-4-6`.
2. **WorkoutAIService** — static methods `generateDailyWorkout()` and `generateDebrief()`. Builds prompts from user profile, recent logs, exercise library, and HealthKit context. Returns parsed `Workout` struct.
3. **ChatAIService** — streaming mid-workout chat. Uses a `---JSON` separator in the AI response to split explanation text (streamed to UI) from structured workout JSON (parsed at end).

### Actor Boundary Pattern

SwiftData models are `@MainActor`. To pass them to async services, views create lightweight `Sendable` snapshot structs (`UserProfileSnapshot`, `WorkoutLogSnapshot`, `ExerciseSnapshot`) immediately before calling services.

### Navigation

`ContentView` is a `TabView` with three tabs: Home, Exercise Library, History. `AppState` (`@Observable`) manages chat drawer overlay visibility. The chat drawer is presented as an overlay on both Home and ActiveWorkout views.

### Seed Data

`SeedData.populate(_:)` fills a `ModelContext` with realistic dummy data (10 exercises, 8 workout logs over 4 weeks with progressive overload, 1 user profile). Auto-runs on every DEBUG launch. `ModelContainer.preview` provides an in-memory seeded container for SwiftUI previews.

## Key Conventions

- `@Observable` for state (not `@StateObject`). Global default actor isolation is `@MainActor`.
- Color system uses `Color(hex: UInt)` extension. Primary palette: `0x0A0A0A`, `0xF5F5F5`, `0x34C759`.
- `JSONExtractor.extractObject()` robustly extracts JSON from AI responses by scanning for balanced braces.
- `ExerciseLibraryService.resolveAndPersistNewExercises()` persists new exercises and resolves their `targetMuscles` at workout-finish time, deduplicating by normalized name.
- `ActiveWorkoutViewModel.applyModifiedWorkout()` reconciles AI-modified workouts with already-logged sets, preserving user progress.

---
> Source: [tuckerritti/lightweight-app](https://github.com/tuckerritti/lightweight-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
