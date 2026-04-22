## bench-mate-android

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

1# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands run from the `bench-mate-android/` directory.

```bash
./gradlew assembleDebug                         # Build debug APK
./gradlew testDebugUnitTest                     # Run all unit tests
./gradlew testDebugUnitTest --tests "app.benchmate.ui.features.bench.BenchViewModelTest"  # Run a single test class
./gradlew installDebug                          # Install to connected device/emulator
```

The repositories library is consumed via Maven Local. If you need an updated version, build and deploy it first from `bench-mate-repositories/`:
```bash
cd ../bench-mate-repositories && ./gradlew deployToMavenLocal
```

## Architecture

Single-screen MVVM app. The only feature is `ui/features/bench/` — a roster view where coaches track which players are on the bench and for how long.

**Data flow:**

```
SQLDelight (SQLite) → PlayerRepository → PlayerUseCase → BenchViewModel → BenchScreen (Compose)
```

- `bench-mate-repositories` is an external KMP library (consumed as `app.benchmate:benchmaterepositories` from Maven Local). It owns all persistence: the SQLDelight schema, `PlayerRepository` interface and `RealPlayerRepository` implementation, and domain models (`Player`, `BenchItem`, `PlayerStatus`).
- `PlayerUseCase` (`common/usecase/`) is the only boundary between the repositories library and the rest of the app. ViewModels never call the repository directly.
- `BenchViewModel` holds all presentation logic. It maps domain models to `PlayerDisplay` / `PlayerStatusDisplay` for the UI, and owns `formatBenchTime(ms: Long): String` as a `companion object` function so it can be unit-tested without Compose.
- `BenchScreen` is a single Composable that observes `BenchViewModel.team: StateFlow<ViewState>`. All user actions (add player, toggle bench, clear bench) are dispatched through the ViewModel.
- Hilt wires everything: `AppModule` provides `PlayerRepository` (singleton), which flows into `PlayerUseCase`, then into `@HiltViewModel BenchViewModel`.

**Bench timing model:**

A player's total bench time splits into two parts:
- `completedBenchMs` — sum of finished `BenchItem` durations (computed in the ViewModel from `startTime`/`endTime` `TimeMark` values).
- `activeBenchStartMs` — wall-clock epoch ms when the current open bench session started (non-null only while the player is actively benched).

`PlayerDisplay.formattedBenchTime` is pre-formatted by the ViewModel for frozen (completed) bench time only. When `activeBenchStartMs != null`, the composable owns a `LaunchedEffect` ticker that recomputes the live total every second and calls `BenchViewModel.formatBenchTime(totalMs)` itself.

## Testing

Unit tests live in `app/src/test/`. The stack is JUnit 4 + Mockito Kotlin + Turbine (Flow assertions) + Truth.

`TestViewModelScopeRule` replaces the main dispatcher with `StandardTestDispatcher` so ViewModel coroutines are controlled in tests.

`BenchTestData` provides canned `Player` lists. When adding players with bench history, use `TimeSource.Monotonic.markNow()` with duration offsets to create `TimeMark` values for `BenchItem`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thereallukesimpson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
