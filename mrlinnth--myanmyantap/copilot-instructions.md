## myanmyantap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Before Starting Any Session

1. Read `docs/CONSTRAINTS.md` — every constraint applies to every file you create or modify. If a task requires deviating from any constraint, stop and ask before proceeding.
2. Read `docs/SESSION-LOG.md` (especially the most recent entries) to understand what has changed since the last session.
3. Pull the latest from `main` before making changes.
4. At the end of each session, append an entry to `docs/SESSION-LOG.md` summarizing what was implemented, anything unexpected, and what remains.

## Project Overview

**Myan Myan Tap** (မြန်မြန်တို့) — a Flutter Android reaction game. A chinlone ball drops from a randomized X position; the player taps before it hits the ground line. One miss ends the game. Ball accelerates with each tap. Score = tap count; best score persisted locally.

**Current status**: planning phase — no source code exists yet. All implementation is defined in `docs/IMPLEMENTATION-PLAN.md` across 7 phases. Begin at Phase 1 unless the session log indicates otherwise.

## Stack

| Item | Version / Value |
|---|---|
| Flutter | 3.38.x (Dart 3.10 bundled — do not pin Dart separately) |
| External packages | `shared_preferences: ^2.3.0` only — no other packages |
| Android target SDK | 35 |
| Android min SDK | 24 |
| NDK | r28 |
| Java | 17 |
| Build output | Android APK only — no iOS, web, or desktop |

## Build Commands

```bash
flutter pub get
flutter run                          # debug on connected device/emulator
flutter build apk --release          # release APK (no flavors, no obfuscation)
flutter test                         # all tests
flutter test test/path/to/test.dart  # single test file
flutter analyze                      # static analysis (strict mode)
```

## Planned File Structure

```
lib/
  main.dart                     # entry point → StartScreen
  screens/
    start_screen.dart
    game_screen.dart
    game_over_screen.dart
  services/
    score_service.dart
docs/
  CONSTRAINTS.md
  MVP-SCOPE.md
  IMPLEMENTATION-PLAN.md
  SESSION-LOG.md
codemagic.yaml                  # CI (Phase 7)
```

## Architecture

### State management
Raw `StatefulWidget` only. No BLoC, Riverpod, Provider, or any state management library.

### Navigation
`Navigator` with `MaterialPageRoute` — no named routes, no router package.

### Game loop
`AnimationController` + `SingleTickerProviderStateMixin` on `GameScreen`'s state. Miss detection uses `AnimationStatus.completed` listener, not a timer (timers drift). Never `setState` inside animation listeners without checking `mounted` first.

### Scoring & speed
- Initial drop duration: 2000ms
- Per-tap multiplier: `max(400, currentDuration * 0.92)` (400ms floor)
- Session score: tap count (held in memory, lost on app close)
- Best score: tap count persisted via `ScoreService`

### Persistence
`ScoreService` uses `SharedPreferencesWithCache` API exclusively — do not use the legacy `SharedPreferences.getInstance()`. Key: `'best_score'`.

### Ball rendering
`BallPainter extends CustomPainter`:
- Burnt orange flat circle (`Color(0xFFD94F2B)`)
- Woven pattern: 3–4 arcs via `canvas.drawArc`
- Motion blur: vertically-smeared radial gradient painted *above* the ball before the circle, scaled to current velocity

### Ground line
`groundY` = 85% of screen height. Miss detection triggers when ball center crosses `groundY`.

### Geometry
- Ball diameter: 48dp fixed
- Spawn X range: `[ballRadius, screenWidth - ballRadius]` (always fully visible)

## Coding Conventions

- `analysis_options.yaml` strict mode: `strict_casts`, `strict_inference`, `strict_raw_types` all `true`
- No implicit `dynamic`; explicit types on all public members
- No `!` without a justification comment
- Extract to a named widget only when reused or when a method exceeds ~40 lines
- `UpperCamelCase` types, `lowerCamelCase` variables/methods, `snake_case` files
- No speculative abstractions — only when there is concrete reuse

## Visual Design

- Background: `Color(0xFFF5F0EB)` on all screens (off-white — not pure white)
- Ball: `Color(0xFFD94F2B)` (burnt orange)
- Ground line: solid black, 2dp
- No images, no textures, no custom fonts, no assets beyond the app icon
- Animations only on the game screen ball drop — all other screens are static

## Explicit Exclusions

No sound, haptics, difficulty levels, leaderboard, accounts, ads, iOS target, additional packages, state management library, custom fonts, or texture-based assets.

---
> Source: [mrlinnth/MyanMyanTap](https://github.com/mrlinnth/MyanMyanTap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
