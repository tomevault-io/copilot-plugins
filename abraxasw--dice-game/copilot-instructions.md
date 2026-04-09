## dice-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Dice Timer** is a SwiftUI iOS app that tracks timing for dice-rolling game rounds across multiple teams. Teams compete to complete rounds, with the app recording when each team rolls their first and last dice.

## Architecture

### Core Data Flow

1. **GameTimer (ObservableObject)** - Central timing mechanism
   - Runs at 0.01s intervals for precise timing
   - Manages start/stop/reset/pause/resume states
   - Published `elapsedTime` drives all UI updates
   - Located: `Dice Game/Models/GameTimer.swift`

2. **Team (Model)** - Team data and round tracking
   - Each team has multiple `RoundTiming` records
   - `RoundTiming` stores `firstDiceTime` and `lastDiceTime` (optional TimeIntervals)
   - Computes statistics: `averageDuration`, `standardDeviation`, `fastestRound`
   - Teams are mutated in-place to record times at specific rounds
   - Located: `Dice Game/Models/Team.swift`

3. **ContentView** - Main game interface
   - Manages game state: `teams`, `currentRound`, timer state
   - Key computed properties:
     - `allTeamsFinished`: checks if all teams have recorded both times for current round
     - `canStartTimer`: prevents starting if round is complete or no teams exist
   - Timer auto-stops when `allTeamsFinished` becomes true
   - Three reset modes: Reset Everything (clears teams), Reset All Rounds (keeps teams), Reset Current Round
   - Located: `Dice Game/ContentView.swift`

### View Hierarchy

```
ContentView (main game screen)
├── Timer Display (round number, elapsed time, status)
├── Progress Indicator (teams completed/total)
├── TeamRowView (for each team)
│   ├── Team info & player count
│   └── Timing buttons (first dice, last dice)
├── Control Panel
│   ├── Start/Stop button
│   ├── Reset button (confirmation dialog)
│   └── Results button (when applicable)
└── Next Round button (when round complete)

AddTeamView (sheet)
└── Team name & player count inputs

ResultsView (sheet)
├── TeamStatsCard (per-team statistics)
└── RoundStatsCard (per-round statistics across teams)
```

### Key State Management Rules

- Teams array is passed as `@Binding` to AddTeamView
- GameTimer is `@StateObject` in ContentView, passed as observed object to child views
- TeamRowView receives callbacks (`onRecordFirst`, `onRecordLast`) to mutate parent state
- All time recordings check and auto-create missing RoundTiming entries
- Last dice time can only be recorded after first dice time for that round

### Testing Structure

- Unit tests: `Dice GameTests/Dice_GameTests.swift`
- UI tests: `Dice GameUITests/` directory

## Building and Testing

This is a standard Xcode project. Common commands:

```bash
# Open in Xcode
open "Dice Game.xcodeproj"

# Build from command line (requires xcodebuild)
xcodebuild -project "Dice Game.xcodeproj" -scheme "Dice Game" -configuration Debug build

# Run tests
xcodebuild test -project "Dice Game.xcodeproj" -scheme "Dice Game" -destination 'platform=iOS Simulator,name=iPhone 15'
```

## Development Notes

- All timing uses `TimeInterval` (Double representing seconds)
- Timer precision is 0.01s (10ms intervals)
- Animations use `.spring()` and `.easeInOut()` for smooth transitions
- Reset operations include brief animation delays for visual feedback
- The app uses `.systemGroupedBackground` as the base color scheme
- SF Symbols are used throughout for icons

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abraxasw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abraxasw)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
