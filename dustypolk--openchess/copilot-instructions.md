## openchess

> This document provides context and guidelines for AI assistants working on this chess game project.

# CLAUDE.md - AI Assistant Guide

This document provides context and guidelines for AI assistants working on this chess game project.

## Project Overview

This is a chess game built with Lua and LÖVE (Love2D) framework. The game features pixel art graphics and implements most standard chess rules.

## Technical Stack

- **Language**: Lua 5.1+
- **Framework**: LÖVE 11.0+
- **Graphics**: Pixel art assets with nearest-neighbor filtering
- **Resolution**: 690x690 window, 180x180 board scaled 3x

## Code Architecture

### Key Variables
- `gameState`: Controls menu/playing states
- `pieces`: 2D array storing piece objects
- `board`: Initial board layout template
- `selectedPiece`: Currently selected piece
- `currentPlayer`: "white" or "black"
- `gameStarted`: Tracks if a game is in progress

### Core Functions
- `initializeBoard()`: Sets up initial piece positions
- `isValidMove()`: Validates piece movements per chess rules
- `isKingInCheck()`: Detects check conditions
- `wouldMoveLeaveKingInCheck()`: Prevents illegal moves
- `getValidMoves()`: Returns all legal moves for a piece

## Asset Information

All assets are in `./assets/`:
- Chess pieces: 13-20px sprites (vary by piece type)
- Board: 180x180px (22.5px per square)
- Scaling: 3x for display (BOARD_SCALE = 3)

## Common Development Tasks

### Adding New Features
When implementing new chess rules (castling, en passant, etc.):
1. Check piece movement in `isValidMove()`
2. Update board state tracking as needed
3. Ensure move validation includes the new rule
4. Test that check detection still works

### Testing Commands
```bash
# Run the game
love .

# No automated tests exist - manual testing required
```

### Code Style
- Use local variables when possible
- Keep functions focused and single-purpose
- Maintain consistent indentation (4 spaces)
- Comment complex chess logic

## Current Limitations to Address

1. **Missing Chess Rules**:
   - Castling (king-side and queen-side)
   - En passant capture
   - Pawn promotion
   - Fifty-move rule
   - Threefold repetition

2. **Game End Conditions**:
   - Checkmate detection
   - Stalemate detection
   - Draw conditions

3. **UI/UX Improvements**:
   - Move history display
   - Captured pieces display
   - Timer/clock system
   - Sound effects
   - Animations for piece movement

4. **Features**:
   - Save/load game
   - PGN export/import
   - AI opponent
   - Online multiplayer
   - Different board themes

## Important Notes

- The game prevents illegal moves that would leave the king in check
- All piece movements follow standard chess rules
- The coordinate system uses 1-based indexing (Lua standard)
- Board squares are 22.5px * 3 = 67.5px in display size
- Menu system preserves game state when using ESC

## Debugging Tips

- Check `pieces` array for current board state
- Use `print()` statements to debug move validation
- Verify `currentPlayer` switches correctly after moves
- Test edge cases like moving into check

## Performance Considerations

- `getValidMoves()` is called every frame when a piece is selected
- Consider caching valid moves if performance becomes an issue
- The game uses simple 2D array lookups, which are fast

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DustyPolk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
