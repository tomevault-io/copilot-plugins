## badook

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Install dependencies
flutter pub get

# Build for web (primary deployment target)
flutter build web --release

# Deploy to Firebase Hosting
firebase deploy --only hosting

# Run linter
flutter analyze

# Run tests
flutter test

# Run single test file
flutter test test/widget_test.dart

# Generate localization files (after modifying .arb files)
flutter gen-l10n
```

## Architecture Overview

This is a multi-game Flutter application targeting web deployment via Firebase Hosting.

### Entry Point & Navigation
- `lib/main.dart` - Main entry with `GameModeSelector` widget for game category selection
- Navigation: MaterialPageRoute-based hierarchical navigation
- Game flow: Category selection → Game selection → Gameplay screen

### Module Structure

**Card Games** (`lib/card_game/`)
- Mighty (마이티): 5-player Korean trick-taking game with bidding, friend declaration, and AI
- Hearts, HiLo, Hula, OneCard, Seven Card Poker
- Pattern: `screens/` + `models/` + `services/` per game

**Board Games** (`lib/board_game/games/`)
- Each game in its own directory: sudoku, tetris, minesweeper, maze, bubble, gomoku, othello, solitaire, baseball, chess, janggi
- Pattern: `*_screen.dart` + models + optional services

**Traditional Games**
- `lib/yutnori/` - Yutnori (윷놀이) with physics-based animation
- `lib/janggi/` - Janggi (장기) Korean chess

**Go (Baduk)** - In `main.dart`
- Life/Death problems (사활) with 30+ problems
- MCTS-based AI opponent

### State Management
- Provider pattern with `ChangeNotifier`
- Game controllers extend `ChangeNotifier` (e.g., `GameController`, `SevenCardController`)
- `StatsService` for player statistics

### Data Persistence
- `SharedPreferences` for game saves and statistics
- JSON serialization for game state
- Single active game save policy (memory efficient)

### Localization
- ARB files in `lib/card_game/l10n/` (ko, en, ja, zh)
- Board game strings in `lib/board_game/l10n/board_game_strings.dart`
- Use `.tr()` extension for translations

### Ad Integration
- `lib/services/web_ad_helper.dart` - Flutter ↔ JavaScript bridge for AdSense
- `web/index.html` - AdSense configuration
- Debug mode uses test ad IDs, release uses production IDs
- Ads shown on game completion screens

### Responsive Design
- Breakpoints: 600px (tablet), 900px (desktop)
- `scaleFactor` pattern for dynamic sizing
- All screens adapt to web/mobile layouts

## Firebase Configuration
- Project: `baduk-game-app`
- Hosting URL: https://baduk-game-app.web.app
- Config files: `firebase.json`, `.firebaserc`

## Key Patterns

**Adding a new game:**
1. Create directory under `lib/board_game/games/` or appropriate location
2. Implement `*_screen.dart` with game UI
3. Add models for game state
4. Add to selection screen navigation
5. Add localization strings

**Game completion ads:**
- Import `web_ad_helper.dart`
- Call `WebAdHelper.showAd()` in completion/result dialog

**AI opponents:**
- Card games use rule-based AI in services
- Go uses MCTS algorithm
- Board games use minimax or custom algorithms

## 오목 AI 패턴 우선순위

`lib/board_game/games/gomoku/gomoku_screen.dart`에서 AI 패턴 감지 우선순위:

1. **순수 연속 3** (`_●●●_`) - 양쪽 열림, 최우선 차단
2. **한칸 건너뛴+연속 3** (`●_●●●`) - 한쪽에 3개 이상 연속
3. **한칸 건너뛴 + 양끝 열림** - 빈칸 채우면 양쪽 열린 4
4. **양끝 막힌 연속 3** (`○_●●●_○`) - 양쪽 빈칸 너머에 상대돌, 낮은 우선순위
5. **일반 한칸 건너뛴 패턴** (`●_●●`)
6. **3x3 가능 위치** - 후순위

### 난이도별 AI 함수
- 쉬움: `_findOpenThree()` - 기본 패턴 감지
- 보통/어려움: `_blockOpenThreeSmartHard()` - 점수 평가로 최적 위치 선택

---
> Source: [mhpark03/badook](https://github.com/mhpark03/badook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
