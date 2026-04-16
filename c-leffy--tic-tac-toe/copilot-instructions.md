## tic-tac-toe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Flutter Tic-Tac-Toe app — Human vs CPU, local play only. Technical challenge following Clean Architecture principles.

## Commands

```bash
flutter pub get                            # Install dependencies (from root)
flutter test packages/game                 # Run game tests
flutter test packages/login                # Run login tests
flutter test packages/game/test/usecases/play_game_test.dart  # Single test file
flutter run                                # Run app (from apps/tic_tac_toe/)
flutter analyze apps/tic_tac_toe           # Analyze the app
flutter analyze packages/game              # Analyze a package
```

## Architecture

Clean Architecture (Ports & Adapters) with four layers inside each feature package (e.g. `packages/game/`):

- **domain/** — Business logic, models, exceptions, strategies. No external dependencies. All models use `Equatable` for value equality. Pure Dart only.
- **application/** — Use cases (one class per action: `StartGame`, `PlayGame`, `PlayIa`). Each use case orchestrates domain logic. Contains `ports/` defining abstract interfaces (e.g. `GameRepository`) that the infrastructure layer implements.
- **infrastructure/** — Implementation details: `persistence/` for repository implementations (e.g. `inmemory/InMemoryGameRepository`), `configuration/` for Riverpod provider wiring (DI).
- **presentation/** — Flutter UI: `view/` for pages, `viewmodel/` for Riverpod AsyncNotifiers (MVI pattern), `widgets/` grouped by domain concept (`board/`, `cell/`, `game/`).

Shared utilities live in `packages/core/` (e.g., `groupBy` extension on `Iterable`).

**Dependency rule**: domain ← application ← infrastructure / presentation. Inner layers never import outer layers. Infrastructure implements application ports.

## Domain Model

- **Game** — Abstract root aggregate holding board, difficulty, and state. Concrete subtypes represent game states: `PlayerTurnGame`, `IaTurnGame`, `HasWinnerGame`, `DrawGame`.
- **Board** — 3×3 grid of `Cell` objects. Win detection via `WinningCondition.hasWinner()`.
- **Cell** — Value object with `x`, `y`, and `Symbol`.
- **Symbol** — Enum: `x`, `o`, `empty`.
- **Difficulty** — Enum: `easy`, `medium`, `difficult`.

## Testing

Tests are in `packages/game/test/` mirroring the source structure:
- `usecases/` — Use case tests (start game, play game, play IA)
- `winning_conditions/` — Board win detection (rows, columns, diagonals)
- `strategy/` — IA strategy tests (random, blocking, minimax)
- `model/` — Game state transition tests
- `presentation/view/` — Page widget tests
- `presentation/viewmodel/` — Notifier tests
- `presentation/widgets/` — Widget tests (board, cell, game)
- `builder/game_builder.dart` — Test builder pattern for constructing `Game` instances

Tests use Given-When-Then structure with descriptive ASCII board representations in test names. Prefer fakes and stubs over mocks.

### Widget Testing Patterns

**Parameterized tests with Dart 3 records:**
```dart
final scenarios = [
  (GameStatusViewModel.playerTurn, 'Your turn'),
  (GameStatusViewModel.victory, 'Victory!'),
];

for (final (status, expectedMessage) in scenarios) {
  testWidgets('Should display $expectedMessage', (tester) async { ... });
}
```

**Faking providers:**
```dart
// For FutureProviders: use overrideWith
startGameUseCaseProvider.overrideWith((ref) async => FakeStartGame())

// For sync providers: use overrideWithValue
analyticsProvider.overrideWithValue(const NoAnalytics())
```

**Faking use cases:**
```dart
class _FakeStartGame implements StartGame {
  @override
  Future<Game> execute(StartGameCommand command) async =>
      PlayerTurnGame(board: Board.generate3x3(), difficulty: command.difficulty);
}
```

**Testing navigation with `ref.listen()`:**
- `ref.listen()` only fires on state *changes*, not the initial state.
- To test navigation, simulate a state change (e.g., `idle` → `gameStarted`).

**Scrolling to off-screen elements:**
```dart
await tester.scrollUntilVisible(find.text('Play'), 100);
await tester.tap(find.text('Play'));
```

**Testing `Visibility` widget:**
```dart
final visibility = tester.widget<Visibility>(find.byType(Visibility));
expect(visibility.visible, isFalse);
```

**Use domain objects in tests** instead of `@visibleForTesting` constructors:
```dart
BoardViewModel createBoard({List<(int x, int y, Symbol symbol)> plays = const []}) {
  var board = Board.generate3x3();
  for (final (x, y, symbol) in plays) {
    board = board.playAt(x, y, symbol);
  }
  return BoardViewModel.from(board);
}
```

## Flutter Rules

The following rules are adapted from the [official Flutter AI rules](https://github.com/flutter/flutter/blob/main/docs/rules/rules.md).

### Dart Style

- Follow [Effective Dart](https://dart.dev/effective-dart).
- Use `PascalCase` for classes, `camelCase` for members/variables/functions/enums, `snake_case` for files.
- Lines should be 80 characters or fewer.
- Write concise, modern Dart code. Prefer functional and declarative patterns.
- Prefer immutable data structures. Widgets (especially `StatelessWidget`) should be immutable.
- Use `const` constructors whenever possible to reduce rebuilds.
- Use arrow syntax (`=>`) for simple one-line functions.
- Use exhaustive `switch` statements/expressions (no `break` needed).
- Use pattern matching and records where they simplify the code.
- Leverage Dart's null safety. Avoid `!` unless the value is guaranteed non-null.
- Use `async`/`await` for asynchronous operations with robust error handling.
- Use custom exceptions for domain-specific error situations.

### Flutter Style

- Favor composition over inheritance for building complex widgets and logic.
- Prefer composing smaller widgets over extending existing ones.
- Use small, private `Widget` classes instead of helper methods that return a `Widget`.
- Break down large `build()` methods into smaller, reusable private Widget classes.
- Avoid performing expensive operations directly within `build()` methods.
- Use `ListView.builder` for long lists (lazy loading).
- Use `LayoutBuilder` or `MediaQuery` for responsive UIs.
- Use `Theme.of(context).textTheme` for text styles.

### Navigation

- Use `go_router` for declarative navigation and deep linking.
- Use `Navigator` for short-lived screens (dialogs, temporary views).

### State Management — MVI with Sealed Classes

This project uses **Riverpod** (`flutter_riverpod`) with the **MVI pattern**, inspired by BLoC but without code generation:

- **Model** = `sealed class` whose subtypes represent every possible screen state. No boolean flags — the type *is* the state.
- **View** = `ConsumerWidget` that uses an exhaustive `switch` on the sealed type. Each branch renders a distinct UI.
- **Intent** = public methods on the `AsyncNotifier` (e.g. `play()`, `startNewGame()`).

**Rules:**
- Define a `sealed class FooState` per screen. Each subtype carries only the data needed for that visual state.
- The Notifier maps domain models to screen states via a `_mapToScreenState()` method. The View never imports domain state types.
- Use `AsyncNotifier` with `AsyncNotifierProvider.autoDispose`. The `AsyncValue` wrapper handles loading/error; the sealed class handles data states.
- Separate ephemeral state (widget-local) from app state (Riverpod providers).

**Navigation via `ref.listen()`:**

For actions that trigger navigation (e.g., starting a game), use a Notifier with state transitions and `ref.listen()` in the view:

```dart
// Notifier with action states
enum HomeState { idle, loading, gameStarted }

class HomeNotifier extends Notifier<HomeState> {
  @override
  HomeState build() => HomeState.idle;

  Future<void> startGame(Difficulty difficulty) async {
    state = HomeState.loading;
    final startGame = await ref.read(startGameUseCaseProvider.future);
    await startGame.execute(StartGameCommand(difficulty: difficulty));
    state = HomeState.gameStarted;
  }
}

// View listens and navigates on state change
ref.listen(homeNotifierProvider, (_, state) {
  if (state == HomeState.gameStarted) {
    ref.read(homeNotifierProvider.notifier).reset();
    context.go('/game');
  }
});
```

This pattern separates business logic (Notifier) from navigation effects (View).

### Code Quality

- Keep functions short, with a single purpose (strive for < 20 lines).
- Avoid abbreviations; use meaningful, descriptive names.
- Write code that is as concise and straightforward as possible.
- Anticipate and handle potential errors. Don't let code fail silently.
- Apply SOLID principles throughout the codebase.

### Documentation

- Use `///` for doc comments on public APIs.
- Start with a single-sentence summary.
- Comment to explain **why**, not **what**. The code should be self-explanatory.
- No useless documentation that restates the obvious.
- No trailing comments.

### Testing Rules

- Follow Given-When-Then (Arrange-Act-Assert) convention.
- Write unit tests for domain logic, use cases, and strategies.
- Write widget tests for UI components.
- Prefer fakes or stubs over mocks.
- Aim for high test coverage.

### Accessibility

- Ensure text contrast ratio of at least 4.5:1 against background.
- Test UI with increased system font size.
- Use `Semantics` widget for descriptive labels on UI elements.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/c-leffy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
