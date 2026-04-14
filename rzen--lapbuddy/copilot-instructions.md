## lapbuddy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LapBuddy is a native iOS swimming lap counter/stopwatch app built with SwiftUI. It helps swimmers track laps, times, and rest intervals during swim sessions.

## Build & Run

This is an Xcode project. Open `LapBuddy.xcodeproj` in Xcode to build and run.

- **Build**: ⌘B in Xcode
- **Run**: ⌘R in Xcode
- **Target Platform**: iOS

## Architecture

### Core State Machine

The app uses a state machine pattern for session management via the `CounterState` enum in `Counter.swift`:

- **reset** → start → **running**
- **running** → pause → **paused** OR add lap → **running** OR auto-complete → **completed**
- **paused** → resume → **running** OR reset → **reset**
- **completed** → reset → **reset**

Each state has associated UI colors defined in the `CounterState.modeColor` computed property.

### Architecture

The app uses an MVVM-like pattern:

- **Counter.swift**: ViewModel (`ObservableObject`) coordinating session logic and state management
  - Manages `CounterState` enum for UI state
  - Handles timer lifecycle (0.1s refresh rate)
  - Creates/updates CoreData Session and Lap entities
  - Tracks timing data (overall time, lap time, pause time)
- **CounterView.swift**: Main stopwatch UI observing Counter state
- **CoreData Entities**: `Session` (1) → (many) `Lap` for persistence

**Important**: There's an `__attic__/` directory containing legacy code. Use the current `Counter.swift` implementation.

### Data Persistence

- **CoreData** for session/lap history storage
- **Model**: `Session` (1) → (many) `Lap` entities, plus `Note` and `Photo` support
- **DataController.swift**: Manages CoreData stack
- **BackupController.swift**: Handles database backups
- **@AppStorage**: Used via `PersistedSettings` for user preferences (lap length, planned laps, feedback settings, etc.)

### View Structure

The app uses a TabView with 4 main tabs:

1. **Counter** (`CounterView`): Main stopwatch interface with:
   - BigDisplayView: Shows lap progress (completed/remaining)
   - CounterStatsView: Current/last/average/fastest lap stats
   - StopwatchView: Animated analog stopwatch with visual feedback
   - State-dependent action buttons (start/pause/resume/lap/reset)

2. **Logbook** (`LogbookView`): Historical session data
3. **Analysis** (`AnalysisView`): Session analytics with charts
4. **Settings** (`SettingsView`): App configuration

### Key Patterns

- **Environment Objects**: `DataController`, `BackupController`, `RestoreIdentity` injected via `ThisApp.swift`
- **@AppStorage via PersistedSettings**: User preferences stored across launches
- **Idle Timer Disabled**: The app prevents screen sleep during use (`UIApplication.shared.isIdleTimerDisabled = true`)
- **Haptic & Sound Feedback**: `HapticFeedback` class for CoreHaptics integration

### Time Formatting

- `ThisApp.formatTimeInterval()`: Centralized time formatting logic
- Shows tenths of seconds for times under 60 seconds
- Format: "MM:SS.t" or "MM:SS" depending on duration

## Important Notes

- Legacy files in `__attic__/` should not be used
- The app uses SwiftUI with CoreData for an iOS-only target
- Timer updates run at 0.1s intervals for smooth display
- Button debouncing (1.0s) prevents accidental double-taps
- always remember to update PUBLIC-RELEASE-NOTES.md when updating release notes
- when making commit messages do not include reference to claude code as being a co-author
- unless explicitly instructed otherwise on every commit also update RELEASE-NOTES.md and PUBLIC-RELEASE-NOTES.md as well as perform a push
- maintain the list of key features in README.md - keep it updated as major features are added

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rzen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rzen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
