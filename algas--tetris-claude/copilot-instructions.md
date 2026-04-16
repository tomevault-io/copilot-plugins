## tetris-claude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based Tetris game implementation using HTML5 Canvas, CSS, and vanilla JavaScript. The game features classic Tetris gameplay with modern web styling.

## Development Setup

This is a client-side web application that requires no build process:

1. Open `index.html` in a web browser to play the game
2. For development, use a local web server for better experience:
   ```bash
   python -m http.server 8000
   # or
   npx serve .
   ```

## Architecture Notes

The game is implemented as a single `Tetris` class with the following key components:

- **Game State Management**: `script.js:Constructor` - Manages board state, score, level, and game status
- **Piece System**: `script.js:generateNewPiece()` - Handles tetromino generation, rotation (`rotateMatrix()`), and movement
- **Board Logic**: `script.js:clearLines()` - Line clearing logic and collision detection (`isValidMove()`)
- **Rendering**: `script.js:draw()` - Canvas-based rendering for game board, pieces, and UI elements
- **Input Handling**: `script.js:setupEventListeners()` - Keyboard controls for movement, rotation, and game controls
- **Game Loop**: `script.js:gameLoop()` - RequestAnimationFrame-based game loop with timer-controlled piece dropping

## File Structure

- `index.html` - Main HTML structure with game canvas and UI elements
- `style.css` - Responsive styling with modern gradient design
- `script.js` - Complete game logic in a single ES6 class

## Game Controls

- ← → : Move piece left/right
- ↓ : Soft drop (faster falling)
- ↑ : Rotate piece
- Space : Hard drop (instant drop)
- P : Pause/resume game

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/algas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
