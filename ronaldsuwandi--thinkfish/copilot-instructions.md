## thinkfish

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ThinkFish is a chess analysis tool that combines Stockfish's calculation power with LLM-generated explanations. Users load PGN games, navigate through moves, and get AI-powered explanations for why Stockfish recommends specific moves.

## Commands

```bash
# Start development server (with hot reload)
npm start

# Install dependencies
npm install
```

The app runs at http://localhost:3000.

## Architecture

### Backend (`src/server.js`)
Express server with two main API endpoints:
- `POST /api/explain-move` - Explains a single move by sending position data (FEN before/after, move played, Stockfish's best move, and continuation) to an LLM
- `POST /api/review` - Generates game reviews (overall, white-focused, or black-focused) by analyzing all moves with their evaluation scores

The server uses the OpenAI SDK but can be configured to different providers (currently pointed at Anthropic API). Requires `OPENAI_API_KEY` in `.env`.

### Frontend (`public/`)
Single-page app with:
- **main.js** - Core application logic:
  - Runs Stockfish in two web workers: one for real-time position analysis (depth 20), one for batch evaluation of all positions
  - Manages game state using chess.js
  - Displays board using chessboard2
  - Renders evaluation chart using Chart.js
  - Converts between UCI and PGN notation for LLM prompts
- **index.html** - UI with chessboard, move list, evaluation chart, and LLM controls

### Data Flow
1. User loads PGN → chess.js parses moves → all positions sent to Stockfish for batch evaluation
2. User navigates to a move → Stockfish analyzes current position at depth 20
3. User clicks "Explain" → frontend packages FEN states, move data, and Stockfish analysis → backend sends to LLM → response displayed

### Key Libraries
- chess.js - Game state and move validation
- chessboard2 - Board visualization with arrow annotations
- stockfish-16.1.js - Chess engine running as web workers (requires COOP/COEP headers)
- Chart.js - Evaluation graph

## Configuration

The server sets Cross-Origin headers for SharedArrayBuffer (required by Stockfish):
```javascript
res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
```

The LLM model can be toggled between GPT-4o and GPT-4o-mini in the UI. The backend model configuration in server.js may need adjustment depending on which API provider you're using.

---
> Source: [ronaldsuwandi/thinkfish](https://github.com/ronaldsuwandi/thinkfish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
