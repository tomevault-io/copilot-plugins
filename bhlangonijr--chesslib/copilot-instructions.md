## chesslib

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build
mvn clean compile package

# Run all tests
mvn test

# Run a single test class
mvn test -Dtest=BoardTest

# Run a single test method
mvn test -Dtest=PerftTest#testPerft1

# Install locally
mvn clean install
```

Java 11 is required. Build system is Maven.

## Architecture

This is a Java chess library (`com.github.bhlangonijr.chesslib`) for legal move generation, FEN/PGN parsing, and game management.

### Core Layer (`chesslib/`)

- **Board** — Central mutable board state. Maintains piece positions as bitboards (64-bit longs), handles move execution/undo via MoveBackup snapshots, generates legal moves, and imports/exports FEN. Uses Zobrist hashing (via XorShiftRandom) for position comparison.
- **Bitboard** — Static bitboard tables and operations. Precomputed attack masks, magic bitboard lookup for sliding pieces.
- **Square, Piece, PieceType, Side, Rank, File** — Enums representing board elements. Square has 64 values (A1–H8) plus NONE. Piece combines PieceType × Side (12 pieces + NONE).
- **CastleRight** — Tracks castling availability per side.

### Move System (`chesslib/move/`)

- **Move** — Immutable value: from-square, to-square, optional promotion piece.
- **MoveList** — Ordered move collection with SAN/FAN string conversion. Uses ThreadLocal Board for efficient notation parsing.
- **MoveGenerator** — Static legal move generation utilities.

### PGN Support (`chesslib/pgn/`)

- **PgnHolder** — Loads/parses/saves PGN files. Supports event listeners for load progress.
- **PgnIterator** — Memory-efficient lazy iterator for large PGN files (uses LargeFile utility).
- **GameLoader** — Parses individual games from PGN text into Game objects.

### Game Model (`chesslib/game/`)

- **Game** — Single chess game with PGN metadata (players, event, result), move list, and variant context.
- **GameContext** — Encapsulates variant-specific rules (castling squares, special moves). Supports chess960 and other variants via VariationType.
- **GameFactory** — Factory for creating Game instances.

### Key Design Decisions

- **Bitboard representation** throughout for O(1) piece lookups and efficient move generation.
- **Event-driven**: Board fires BoardEvent to BoardEventListener on moves/loads, enabling GUI integration.
- **Full undo support**: Every doMove() stores a MoveBackup; undoMove() restores complete state.
- **Zobrist incremental hashing**: Board maintains a hash key updated incrementally on each move for fast position comparison.

## Dependencies

- Runtime: Apache Commons Lang 3
- Test: JUnit 4

## Test Resources

PGN test files are in `src/test/resources/` — includes standard games, edge cases (en passant, promotion, castling), large files for performance testing, and error condition files.

## GUIDELINES
## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---
> Source: [bhlangonijr/chesslib](https://github.com/bhlangonijr/chesslib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
