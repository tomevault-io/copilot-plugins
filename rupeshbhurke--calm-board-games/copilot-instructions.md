## calm-board-games

> Plugin-based game architecture rules


# Architecture Rules

## Plugin-Based Game Architecture

### Module Structure

Every game MUST follow this structure:

```
lib/lib/games/<game_id>/
├── <game_id>_module.dart      # Implements GameModule
├── <game_id>_screen.dart      # Main UI widget
└── logic/
    └── <game_id>.dart         # Pure game logic
```

### GameModule Interface

All games MUST implement `GameModule`:

```dart
class MyGameModule implements GameModule {
  const MyGameModule();

  @override
  GameMetadata get metadata => const GameMetadata(
    id: 'my_game',           // snake_case
    title: 'My Game',        // Display name
    tagline: 'Description',  // Short tagline
    category: GameCategory.puzzle,
  );

  @override
  Widget buildGameScreen() => const MyGameScreen();
}
```

### Registration

Games MUST be registered in `lib/lib/games/registry/game_registry.dart`:

```dart
modules: const [
  SlidingPuzzleModule(),
  Game2048Module(),
  // Add new modules here
],
```

### DO NOT

- Create games outside `lib/lib/games/`
- Use reflection or dynamic registration
- Skip the `GameModule` interface
- Register games in multiple places

---

## UI and Logic Separation

### Logic Files

Files in `logic/` directories:
- MUST be pure Dart (no Flutter imports)
- MUST NOT import `package:flutter/*`
- MUST be testable without Flutter test harness
- SHOULD use immutable state classes

### UI Files

Screen and widget files:
- MAY import Flutter and design tokens
- MUST import logic from `logic/` subdirectory
- MUST NOT contain game rule logic
- SHOULD use `StatefulWidget` for game state

### Boundary Check

Before committing, verify:
1. No `package:flutter` in any `logic/` file
2. All game rules are in `logic/` files
3. UI only renders and handles input

---

## Shared Code

### When to Extract

Extract to shared utilities when:
- Same pattern appears in 2+ games
- Logic is generic (grid operations, RNG, etc.)
- UI component is reusable (dialogs, score displays)

### Where to Place

| Type | Location |
|------|----------|
| Shared UI | `lib/lib/ui/` |
| Shared logic | `lib/lib/engine/` (create if needed) |
| Design tokens | `lib/lib/design/` |

### DO NOT

- Duplicate functionality between games
- Create game-specific code in shared directories
- Modify shared code without checking all usages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rupeshbhurke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
