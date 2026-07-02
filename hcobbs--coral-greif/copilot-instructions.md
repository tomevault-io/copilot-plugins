## coral-greif

> *Copyright (C) 2024 T. Hunter Cobbs. Licensed under GPLv3.*

# CLAUDE.md - Coral Greif Project Instructions

*Copyright (C) 2024 T. Hunter Cobbs. Licensed under GPLv3.*

This file provides project-specific guidance for Claude Code when working on Coral Greif.

**Note:** Always reference the parent `~/dev/CLAUDE.md` for identity, personality, and general coding standards.

---

## Project Overview

Coral Greif is an iOS Battleship-style game with WWII Pacific theater theming. Native Swift, UIKit, SpriteKit. No third-party dependencies.

---

## Build Commands

```bash
# Generate Xcode project
xcodegen generate

# Build
xcodebuild -scheme CoralGreif -destination 'platform=iOS Simulator,name=iPhone 17'

# Run tests (required before any commit)
xcodebuild test -scheme CoralGreif -destination 'platform=iOS Simulator,name=iPhone 17'

# Clean build
xcodebuild clean -scheme CoralGreif
```

---

## Code Architecture

### Directory Structure

```
CoralGreif/
├── Core/
│   └── Models/           # Data models (immutable where possible)
├── Engine/               # Game logic, AI, turn management
├── Persistence/          # Save/load via Codable + UserDefaults
├── UI/                   # UIKit views and view controllers
└── Visualization/        # SpriteKit scenes and nodes
```

### Design Principles

1. **Immutability First**
   - Models should be structs, not classes
   - Use `let` by default, `var` only when mutation is necessary
   - Mutation via copy-on-write where applicable

2. **No Polling**
   - All interfaces must be interrupt-driven
   - Use delegates, closures, Combine, or async/await
   - Timer callbacks are acceptable for turn limits
   - Never spin-wait or poll state

3. **Test Everything**
   - 100% test coverage for models
   - All edge cases must have tests
   - Tests run in CI; failures block merges

4. **Security at Boundaries**
   - Validate all external input
   - Failable initializers for constrained types
   - Never trust data from files, network, or user input

---

## Swift Conventions

### Naming

- Types: `PascalCase`
- Functions, variables: `camelCase`
- Constants: `camelCase` (not `SCREAMING_CASE`)
- Protocols describing capability: `-able`, `-ible` suffix (e.g., `Codable`)
- Protocols describing type: noun (e.g., `GameEngine`)

### Formatting

- **4 spaces** for indentation (no tabs)
- Line length: 100 characters soft limit, 120 hard limit
- Opening braces on same line
- One blank line between functions
- No trailing whitespace

### Style Rules

- **No ternary operators.** Use if/else for clarity.
- **No force unwrapping** except in tests or after explicit nil checks.
- **No implicitly unwrapped optionals** except for IBOutlets.
- **Guard early.** Validate preconditions at function start.
- Prefer `forEach` only for side effects; use `map`/`filter` for transformations.

### Documentation

- All public types and functions require documentation comments
- Use `///` for documentation, `//` for implementation notes
- Document parameters, return values, and thrown errors
- Keep docs concise; the code should be readable on its own

---

## Game-Specific Rules

### Coordinate System

- Grid is 10x10 (0-9 for both row and column)
- Row 0 is top, Row 9 is bottom
- Column 0 is left (A), Column 9 is right (J)
- Display format: Letter + Number (e.g., "A1", "J10")

### Ship Sizes

| Type | Size |
|------|------|
| Carrier | 5 |
| Battleship | 4 |
| Cruiser | 3 |
| Submarine | 3 |
| Destroyer | 2 |

### Game Flow

1. **Setup Phase:** Both players place ships
2. **Battle Phase:** Players alternate attacks
3. **Finished Phase:** One player sinks all opponent ships

Turn timer: 20 seconds default. Timeout results in random valid attack.

### AI Difficulty Levels

- **Ensign:** Random targeting
- **Commander:** Hunt-and-target algorithm
- **Admiral:** Probability-based targeting with pattern analysis

---

## Testing Standards

### Test Naming

```swift
func test<MethodName>_<Condition>_<ExpectedResult>()

// Examples:
func testPlaceShip_OutOfBounds_ReturnsFailure()
func testReceiveAttack_AlreadyAttacked_ReturnsError()
func testAllShipsSunk_AllShipsSunk_ReturnsTrue()
```

Or simpler naming for straightforward tests:

```swift
func testShipCreation()
func testCodableRoundTrip()
```

### Test Organization

- One test file per source file
- Mirror source directory structure in test directory
- Group related tests with `// MARK: -` comments
- Helper functions should be `private` within test file

### Required Test Coverage

- All model initializers
- All public functions
- All computed properties
- Error cases and edge cases
- Codable round-trip serialization
- Equatable conformance

---

## Git Workflow

### Branch Naming

- `feature/<description>` for new features
- `bugfix/<description>` for bug fixes
- `docs/<description>` for documentation
- `refactor/<description>` for refactoring

### Commit Messages

Use `[LLM-ARCH]` label for AI-assisted development:

```
[LLM-ARCH] Brief description of change

Longer explanation if needed. Focus on why, not what.
The diff shows what changed.
```

### Pre-Commit Checklist

- [ ] All tests pass
- [ ] No compiler warnings
- [ ] Code follows style guide
- [ ] New code has documentation
- [ ] No debug code or print statements left behind

---

## Common Pitfalls

### Don't

- Use `class` when `struct` suffices
- Force unwrap optionals in production code
- Ignore Result failure cases
- Use polling or busy-waiting
- Add dependencies without discussion
- Commit failing tests
- Use em dashes in documentation

### Do

- Validate at boundaries
- Test edge cases
- Use guard for early returns
- Document public interfaces
- Keep functions small and focused
- Make illegal states unrepresentable

---

## Resources

- [PLAN.md](PLAN.md): Development roadmap and phase details
- [Game Manual](Documentation/GameManual.md): User-facing documentation
- [Privacy Policy](Documentation/PrivacyPolicy.md): App Store privacy requirements

---
> Source: [hcobbs/coral_greif](https://github.com/hcobbs/coral_greif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
