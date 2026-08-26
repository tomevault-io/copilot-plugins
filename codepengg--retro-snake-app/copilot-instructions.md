## retro-snake-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Read this first

A detailed knowledge base lives in **`.ai/`**. Start with **`.ai/memory.md`** (the
condensed operating context), then the relevant `.ai/context/*.md` for whichever
layer you are touching. `.ai/planning/tasks.md` holds the prioritised backlog with
analysis already done — check it before reporting a "new" issue.

⚠️ **`README.md` is partly stale.** It documents a `20x20` grid as static consts in
`game_state.dart` (the grid is now constructor-injected and device-dependent),
lists sound files that do not exist (`eat.wav`, `game_over.wav`, `start.wav`), and
tells you to uncomment the `fonts:` block in `pubspec.yaml` (already done). Trust
the code and `.ai/` over the README.

## Commands

```bash
flutter pub get

flutter analyze                                  # must stay at "No issues found"
flutter test                                     # 30 tests
flutter test test/game_state_test.dart           # single file
flutter test --plain-name "rejects a 180-degree reversal"   # single test by name

flutter run -d <device>
flutter build apk --debug
flutter build apk --release       # requires android/key.properties (untracked)
flutter build appbundle --release # Play Store artifact
```

`flutter` is a shell function wrapping fvm. It prints harmless
`_fvm_project_root_flutter: command not found` lines before real output — ignore
them.

## Architecture

Layered-by-type widget app, ~2,650 lines in `lib/`. **No state-management package,
no DI container, no networking, no database, no router package, no CI.** These are
deliberate decisions recorded in `.ai/decisions/adr-001..003.md`, appropriate for a
single-feature offline game. Do not add any of them without approval.

```
lib/
├── main.dart              MaterialApp · ThemeData · Routes constants
├── game/
│   ├── game_state.dart    DOMAIN — the only clean boundary
│   ├── snake_game.dart    loop + input + audio + lifecycle + layout
│   └── snake_painter.dart Canvas rendering
└── ui/
    ├── app_colors.dart    the only design-token file
    ├── start_screen.dart  (755 lines)
    ├── game_over_screen.dart (688 lines)
    └── common/retro_button.dart  the only extracted shared widget
```

### The one hard rule

**`lib/game/game_state.dart`'s entire import list is `import 'dart:math';`.** No
Flutter, no plugins, no I/O. That property is why 23 unit tests run without a
widget tree, and it is the only real architectural seam in the project. If you
need change notification, wrap it — `GameController extends ChangeNotifier` owning
a plain `GameState` — rather than importing Flutter into the domain file.

Business logic belongs in `GameState`. Widgets orchestrate; they do not compute
game rules.

### Data flow

`Timer.periodic(200ms)` → `GameState.update()` → `setState(() {})` → rebuild →
new `SnakePainter`. Direction input is buffered one tick: `setDirection` writes
`_nextDirection`, `update()` commits it to `_direction`. The 180°-reversal guard
compares against the **committed** direction, which is what prevents a two-step
self-kill.

### Painter contract

`GameState` is mutated in place, so a painter comparing it by reference always
reports "unchanged". `SnakePainter` snapshots value fields in its constructor and
compares those in `shouldRepaint`, plus `super(repaint: animation)` so animations
repaint without `setState`. **Any painter reading `GameState` must copy this
pattern.**

### Persistence

One key: `high_score` (`int`) in `SharedPreferences`. That is the entire persisted
state of the app.

**`GameOverScreen` is the single writer.** `SnakeGame` used to write it too, which
made the "is this a new best?" comparison always false and left the ★ NEW HIGH
SCORE ★ banner structurally unreachable. Three widget tests enforce this — adding a
second writer will fail them, intentionally. New-best is **strictly greater**; a
tie does not celebrate.

### Navigation

`Navigator` 1.0 with three flat named routes. Use the `Routes` constants in
`main.dart`, never string literals. **Return home with
`popUntil(ModalRoute.withName(Routes.home))`**, never `pushReplacementNamed('/')` —
the latter stacks a second `StartScreen` and leaks its three `AnimationController`s,
timer, and `AudioPlayer` once per round.

## Deliberate behaviours — do NOT "fix" these

Each has a source comment. Changing any is a product decision, not a bug fix.

1. **Entering the tail cell is fatal**, though it is vacated that same tick.
   Diverges from classic Snake.
2. **Resume does not auto-unpause** — the player may not be looking at the screen.
3. **`dispose()` is the sole owner of `AudioPlayer` lifetime.** Lifecycle callbacks
   call `stop()`, never `dispose()`; disposing on background previously left the
   player unusable and double-disposed it.
4. **Random geometry is generated once in `initState`** into a `late final` field,
   never in `build` or an `AnimatedBuilder` — `GameOverScreen`'s 1 Hz countdown
   `setState` would reshuffle it. Regression-tested.
5. **`_endGame()` delays navigation 2 s (death) / 1 s (win)** so the crash sound
   and glitch animation play. Navigation must never move back into `build()`.

## Testing gotchas

- **`pumpAndSettle()` never settles** on `StartScreen` or `GameOverScreen` — both
  run infinite `repeat()` animations, and `GameOverScreen` additionally runs a 10 s
  auto-return countdown that navigates the screen away mid-test. Use `pump()` with
  explicit durations.
- **No fake exists for `audioplayers`**, so `SnakeGame` cannot be pumped in a
  widget test. This is the largest coverage gap. Do not fabricate a passing test
  for it — say the gap out loud.
- `MaterialApp` cannot take both `home:` and a `/` entry in `routes:`. Use
  `onGenerateInitialRoutes` to build a real stack when testing `popUntil`.
- `SharedPreferences.setMockInitialValues({})` in `setUp`.
- Package is `retro_snake`, **not** `retro_snake_app` — a wrong import here is what
  made the original suite fail to compile.

## UI conventions

Palette in `AppColors`; semantics are **green = the player, purple = the
goal/record, blue = snake body & secondary actions, gold = celebration, red =
failure**.

- Colours from `AppColors` — never a raw `Color(0x…)` in a widget.
- **Every neon element gets a matching-hue glow** (`BoxShadow` or
  `MaskFilter.blur`). Most consistently applied rule in the codebase.
- `withValues(alpha:)` — `withOpacity` was fully removed, do not reintroduce it.
- **Do not add `fontFamily` to new `TextStyle`s** — `ThemeData` supplies
  `PressStart2P`.
- `FontWeight.normal` only; the pixel face has one weight.
- Scores render as `value.toString().padLeft(6, '0')`.

⚠️ No spacing/type/radius tokens exist, and **three sizing systems run in parallel**
(`flutter_screenutil`, MediaQuery ratios, hardcoded breakpoints). Do not add a
fourth. Convergence is tracked as task T-10.

## Known traps

| Trap | Detail |
|---|---|
| `GameState` is created inside `build()` | Any constraint change (keyboard, split-screen) resets the run to score 0. Known bug, task T-06, `TODO` at `snake_game.dart:380` |
| Grid is device-dependent | 20 columns fixed; rows = `clamp(20, 50)` from available height. Scores are not comparable across devices |
| `.env` holds a keystore password and is tracked in git | **Unremediated.** See `SECURITY_REMEDIATION.md` |
| `origin/feature/movement-button` is diverged | Removes `flutter_screenutil`; overlaps planned work. Review before any sizing or controls refactor |
| Portrait is locked in `main.dart` | Makes `StartScreen._buildLandscapeLayout()` dead code on mobile, despite `Info.plist` declaring landscape |
| Release build requests zero Android permissions | Preserve this — no plugin currently drags one in |
| Grep false positives | `grep -c http pubspec.yaml` → 5 and `grep -c dio` → 2, from doc URLs and the word `au`**dio**`players`; `hive` matches `arc`**hive**`d`. There is no networking or database |

---
> Source: [codepengg/retro_snake_app](https://github.com/codepengg/retro_snake_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
