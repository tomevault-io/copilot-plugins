## hearts

> A pure Swift package implementing the complete game logic for Hearts. This package is designed to be UI-agnostic and can be consumed by any Apple platform application (iOS, macOS, tvOS, watchOS) or command-line tools.

# Hearts Game Engine

A pure Swift package implementing the complete game logic for Hearts. This package is designed to be UI-agnostic and can be consumed by any Apple platform application (iOS, macOS, tvOS, watchOS) or command-line tools.

## Project Overview

This is a Swift Package Manager (SPM) package that serves as the core engine for a Hearts card game. It contains all game rules, state management, AI logic, and validation—completely independent of any UI framework.

### Design Principles

- **Zero UI Dependencies**: No UIKit, AppKit, SwiftUI, or any presentation framework imports
- **Pure Swift**: Uses only Swift standard library and Foundation where necessary
- **Protocol-Oriented**: Favors protocols and composition over inheritance
- **Testable by Design**: All components are injectable and mockable
- **Platform Agnostic**: Runs on any Apple platform or Linux

### Target Test Coverage: 95%+

Every public API must have comprehensive test coverage. Edge cases, error conditions, and boundary scenarios must be tested.

## Architecture

```
Sources/HeartsEngine/
├── Models/           # Core data types (Card, Suit, Rank, Player, Hand, Trick, etc.)
├── Rules/            # Game rules and validation logic
├── State/            # Game state management and transitions
├── AI/               # Computer player strategies
├── Scoring/          # Score calculation and tracking
└── Events/           # Game events and notifications (delegate/callback patterns)

Tests/HeartsEngineTests/
├── Models/           # Model unit tests
├── Rules/            # Rules validation tests
├── State/            # State transition tests
├── AI/               # AI behavior tests
├── Scoring/          # Scoring calculation tests
├── Integration/      # Full game flow tests
└── Mocks/            # Test doubles and fixtures
```

## Code Style Guidelines

### Naming Conventions

- Types: `PascalCase` (e.g., `GameState`, `CardValidator`)
- Functions/Methods: `camelCase` (e.g., `playCard()`, `calculateScore()`)
- Constants: `camelCase` (e.g., `maxHandSize`, `pointsToLose`)
- Protocols: Noun or adjective describing capability (e.g., `CardPlayable`, `ScoreCalculating`)

### Swift Conventions

- Use `struct` for value types (Card, Hand, Score)
- Use `class` only when reference semantics are required (GameEngine, AIPlayer)
- Use `enum` for finite sets (Suit, Rank, GamePhase)
- Prefer `let` over `var`
- Use `guard` for early exits
- Avoid force unwrapping (`!`) except in tests with known data

### Documentation

- All public APIs must have documentation comments
- Use `///` for documentation
- Include parameter descriptions and return value explanations
- Add code examples for complex APIs

```swift
/// Attempts to play a card from a player's hand.
/// - Parameters:
///   - card: The card to play
///   - player: The player attempting the play
/// - Returns: The resulting game state after the play
/// - Throws: `InvalidPlayError` if the play violates game rules
func play(card: Card, by player: Player) throws -> GameState
```

## Testing Requirements

### Test Structure

- One test file per source file (e.g., `Card.swift` → `CardTests.swift`)
- Use descriptive test names: `test_methodName_condition_expectedResult()`
- Group related tests using `// MARK: -` comments
- Each test should test one behavior

### Test Naming Convention

```swift
func test_playCard_whenPlayerDoesNotHaveSuit_allowsAnyCard() { }
func test_calculateScore_withQueenOfSpades_returns13Points() { }
func test_passingPhase_withThreeCards_transitionsToPlayingPhase() { }
```

### What to Test

- All public methods and properties
- All error conditions and edge cases
- State transitions
- Boundary conditions (empty hands, full tricks, game end)
- AI decision-making logic
- Score calculations including "shooting the moon"

### Test Helpers

Create reusable test fixtures and builders:

```swift
// In Tests/Mocks/
extension Card {
    static let aceOfSpades = Card(suit: .spades, rank: .ace)
    static let queenOfSpades = Card(suit: .spades, rank: .queen)
}

extension Hand {
    static func mock(cards: [Card]) -> Hand { ... }
}
```

## Hearts Game Rules Reference

### Basic Rules

- 4 players, each receives 13 cards
- Players pass 3 cards before each hand (left, right, across, no pass—rotating)
- Player with 2 of clubs leads first trick
- Must follow suit if possible
- Hearts cannot be led until "broken" (played on another suit)
- No points on first trick (no hearts or Queen of Spades)

### Scoring

- Each heart: 1 point
- Queen of Spades: 13 points
- Shooting the Moon: Player takes all hearts + Queen of Spades = 0 points, others get 26
- Game ends when a player reaches 100 points (configurable)
- Lowest score wins

### Valid Play Rules

1. Must follow lead suit if able
2. Cannot lead hearts until broken (unless only hearts remain)
3. Cannot play points on first trick (unless no choice)
4. 2 of clubs must lead the first trick of each hand

## Common Tasks

### Adding a New Feature

1. Write failing tests first (TDD approach)
2. Implement the minimal code to pass tests
3. Refactor while keeping tests green
4. Ensure no UI framework imports
5. Update documentation

### Running Tests

```bash
swift test
swift test --enable-code-coverage
```

### Generating Coverage Report

```bash
swift test --enable-code-coverage
xcrun llvm-cov report .build/debug/HeartsEnginePackageTests.xctest/Contents/MacOS/HeartsEnginePackageTests -instr-profile .build/debug/codecov/default.profdata
```

## Error Handling

Use typed errors for game rule violations:

```swift
enum InvalidPlayError: Error {
    case notPlayersTurn
    case cardNotInHand
    case mustFollowSuit(required: Suit)
    case heartsNotBroken
    case cannotPlayPointsOnFirstTrick
    case mustLeadTwoOfClubs
}
```

## State Management

The game engine should be the single source of truth. State changes should be:

- Immutable where possible (return new state rather than mutating)
- Validated before applying
- Observable via delegate/callback pattern (no Combine or other reactive frameworks)

```swift
protocol GameEngineDelegate: AnyObject {
    func gameEngine(_ engine: GameEngine, didTransitionTo state: GameState)
    func gameEngine(_ engine: GameEngine, playerDidPlay card: Card, by player: Player)
    func gameEngine(_ engine: GameEngine, didCompleteTrick trick: Trick, winner: Player)
    func gameEngine(_ engine: GameEngine, didCompleteHand scores: [Player: Int])
    func gameEngine(_ engine: GameEngine, didEndGame winner: Player)
}
```

## AI Implementation

AI players should conform to a strategy protocol:

```swift
protocol AIStrategy {
    func selectCardsToPass(from hand: Hand, direction: PassDirection) -> [Card]
    func selectCardToPlay(from hand: Hand, in trick: Trick, gameState: GameState) -> Card
}
```

Implement multiple difficulty levels:

- `RandomAIStrategy`: Random valid plays (for testing)
- `BasicAIStrategy`: Simple heuristics (avoid points)
- `AdvancedAIStrategy`: Strategic play (card counting, shooting the moon detection)

## Dependencies

**Allowed:**
- Swift Standard Library
- Foundation (minimal use—avoid platform-specific APIs)

**Not Allowed:**
- UIKit, AppKit, SwiftUI, WatchKit
- Combine, RxSwift, or reactive frameworks
- CoreGraphics, CoreAnimation, or graphics frameworks
- Any third-party packages (keep it dependency-free)

## Checklist Before Committing

- [ ] All tests pass (`swift test`)
- [ ] No UI framework imports
- [ ] Public APIs are documented
- [ ] Test coverage ≥ 95%
- [ ] No force unwraps in production code
- [ ] Error cases are handled with typed errors
- [ ] Code follows naming conventions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HassanSE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
