## pokerkit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PokerKit is a Swift package for poker hand evaluation and equity calculation. It uses a lookup table-based algorithm with a large binary data file (HandRanks.dat, ~130MB) to efficiently evaluate poker hands and calculate hand equity against multiple opponents.

## Common Commands

### Building
```bash
swift build -v
```

### Testing
```bash
# Run all tests
swift test -v

# Run all tests with xUnit output (for CI)
swift test -v --xunit-output test-results/test.xml --parallel

# Run a specific test
swift test --filter "HandEvaluatorTests"
```

### Linting
```bash
# SwiftLint runs automatically on PRs via GitHub Actions
# To run locally (if SwiftLint is installed):
swiftlint
```

## Architecture

### Core Design Pattern

The package uses a **Protocol-Oriented Design** with the `HandEvaluator` protocol as the main abstraction. Hand evaluation is implemented through a lookup table strategy that processes cards incrementally.

### Key Components

1. **HandEvaluator Protocol** (`Protocols/HandEvaluator.swift`)
   - Defines the interface for all hand evaluation strategies
   - Two methods: `evaluate(card:handle:)` for incremental evaluation and `evaluate(handle:)` for final ranking

2. **LookupTableHandEvaluator** (`Implementation/LookupTableHandEvaluator.swift`)
   - Concrete implementation using a pre-computed lookup table
   - Loads a 130MB binary data file (`HandRanks.dat`) from Resources
   - Supports 5, 6, and 7 card hands
   - Uses `HandHandle` for incremental card-by-card evaluation

3. **EquityCalculator Protocol** (`Protocols/EquityCalculator.swift`)
   - Defines the interface for equity calculation strategies
   - Two methods: `calculateEquity(hand:board:numOpponents:)` for unknown opponents and `calculateEquity(hand:board:opponentHands:)` for known opponent hands

4. **DefaultEquityCalculator** (`Implementation/DefaultEquityCalculator.swift`)
   - Concrete implementation using combinatorial analysis and Monte Carlo simulation
   - Uses the HandEvaluator to evaluate all possible outcomes
   - Switches to Monte Carlo (1M iterations) when combinations exceed 1M
   - Handles incomplete boards (0-5 community cards)
   - Requires swift-algorithms package for efficient combination generation

5. **Models**
   - `Card`: Represents a playing card with `rank` and `suit`, has an `id` computed property used for lookup table indexing
   - `HandHandle`: Tracks evaluation state as cards are added incrementally (contains `value` for lookup and `hand` array)
   - `HandRank`: Final evaluation result with `category` (high card, pair, etc.) and `rank` for comparison
   - `Rank` and `Suit`: Standard playing card enumerations

6. **Factory Pattern** (`PokerKit.swift`)
   - `PokerKit.lookupTableHandEvaluator()` provides the standard evaluator instance
   - `PokerKit.defaultEquityCalculator(evaluator:)` provides the equity calculator (defaults to using lookup table evaluator)

### How Hand Evaluation Works

1. Start with `HandHandle.empty` (value: 53)
2. For each card, call `evaluator.evaluate(card:handle:)` which:
   - Looks up the next state using `lookupTable[handle.value + card.id]`
   - Returns a new `HandHandle` with updated value and hand array
3. After 5-7 cards, call `evaluator.evaluate(handle:)` to get the final `HandRank`
4. HandRank encodes both category (hand type) and rank (strength within category) in a single value for comparison

### How Equity Calculation Works

1. **Unknown Opponents Mode** (`calculateEquity(hand:board:numOpponents:)`):
   - Builds a deck excluding known cards (hero's hand + board)
   - Generates all possible combinations of remaining cards for board completion and opponent hands
   - For each combination, evaluates hero's hand vs opponent hands
   - Switches to Monte Carlo sampling if combinations exceed 1M

2. **Known Opponents Mode** (`calculateEquity(hand:board:opponentHands:)`):
   - Used when opponent hands are known (e.g., all-in showdown situations)
   - Builds a deck excluding all known cards
   - If board is incomplete, enumerates all possible board completions
   - For each board completion, compares hero's hand rank against opponent hand ranks
   - Returns exact equity (0.0 for loss, 1.0 for win, 0.5 for chop in heads-up, 1/n for n-way chop)

3. **Monte Carlo Simulation**:
   - Triggered when total combinations exceed 1,000,000 (defined by `maxCombinationsCount`)
   - Randomly samples 1M scenarios instead of exhaustive enumeration
   - Provides fast approximation for complex scenarios (e.g., preflop multiway with many opponents)

4. **Equity Calculation Details**:
   - Equity is calculated as the probability of winning or chopping
   - For a win, equity = 1.0 divided by the number of players with the same best hand rank
   - Uses HandEvaluator to rank all hands and determine winners
   - Properly handles chops (multiple players with identical hand ranks)

### Resources and Git LFS

The `HandRanks.dat` file is tracked with Git LFS due to its size. CI/CD workflows include `git lfs checkout` to ensure the file is available during builds and tests.

## Swift Package Configuration

- Minimum Swift version: 6.2
- Supports macOS 10.15+, iOS 13+, watchOS 6+, tvOS 13+
- Uses StrictConcurrency experimental feature
- Resources are copied (not processed) to support Bundle.module access
- SwiftLint disabled rule: `identifier_name`
- Dependencies:
  - swift-algorithms: Used for efficient combination generation in equity calculations

## Testing Framework

Uses Swift Testing (not XCTest). Key patterns:
- `@Suite` for test grouping
- `@Test` for individual tests
- `#expect` for assertions
- `#require` for non-optional unwrapping in tests

Test suites include:
- `HandEvaluatorTests`: Tests for all hand categories and edge cases
- `EquityCalculatorTests`: Tests for equity calculation with known/unknown opponents, complete/incomplete boards, and various hand matchups

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ittna) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
