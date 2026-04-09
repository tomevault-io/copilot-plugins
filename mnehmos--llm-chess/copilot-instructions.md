## llm-chess

> LLM Chess is an LLM-vs-LLM chess arena, tournament gauntlet, and benchmark app. Built with React 18, TypeScript, Vite, Zustand, and chess.js. Optional Tauri desktop wrapper.

# CLAUDE.md — Project Knowledge for LLM Chess

## Project Overview

LLM Chess is an LLM-vs-LLM chess arena, tournament gauntlet, and benchmark app. Built with React 18, TypeScript, Vite, Zustand, and chess.js. Optional Tauri desktop wrapper.

## Tech Stack

- **Frontend:** React 18 + TypeScript + Vite
- **State:** Zustand with `persist` middleware (localStorage)
- **Chess:** chess.js for move validation/board state
- **Stockfish:** Web Worker-based engine for eval and AI opponent
- **LLM Providers:** OpenRouter, OpenAI, Ollama (local), Codex (device auth)
- **TTS:** Local Python server or cloud API (Qwen)
- **Desktop:** Optional Tauri bridge (`src/tauri-bridge.ts`)

## Architecture

### Source Layout

```
src/
  engine/         # Core game logic: runtime, turn handling, game loop
    runtime.ts        # Main game runtime (~982 lines, handles all 5 player types)
    orchestrator-runtime.ts  # Multi-board orchestrator mode
    turn-runtime.ts   # Per-turn context building and prompt assembly
    simul-runtime.ts  # Simultaneous multi-board games
    types.ts          # Core type definitions (PlayerConfig, GameState, MoveRecord)
    events.ts         # Event type definitions (GameEvent discriminated union)
    reducer.ts        # Event -> state reducer
    transitions.ts    # Attack injection scheduling and timing
    attacks.ts        # Attack channel implementations (prompt injection, FEN corruption, etc.)
    advisor.ts        # Stockfish-based move advisor
    fog-of-war.ts     # Fog of war board masking
    constraints.ts    # Wall-clock and token constraints
    response-toolkit.ts  # Graduated response tools for LLM players
    oracle-player.ts  # Stockfish-guided LLM oracle mode
  llm/            # LLM provider clients and prompt construction
    client.ts         # Factory: createLLMClient(provider, apiKey, ...)
    openai.ts         # OpenAI provider client
    openrouter.ts     # OpenRouter provider client
    ollama.ts         # Ollama (local) provider client
    openai-codex-bridge.ts  # Codex provider bridge
    prompts.ts        # System/user prompt construction
    parser.ts         # LLM response parsing (JSON + regex fallbacks)
    model-capabilities.ts   # Model feature detection, downgrades, token budgets
  store/          # Zustand stores
    gameStore.ts      # Single game state management
    tournamentStore.ts    # Tournament orchestration (~1474 lines, largest store)
    benchmarkStore.ts # Benchmark data collection and aggregation
    settingsStore.ts  # User settings, API keys, provider config
  components/     # React UI components
    TournamentProgress.tsx  # Tournament game display (~1141 lines)
    TournamentSetup.tsx     # Tournament configuration (~960 lines)
    AdvancedPlayerConfig.tsx # Per-player advanced settings
    GameControls.tsx        # Single game setup and controls
    GameLayout.tsx          # Board + sidebar layout
    Board.tsx               # Chessboard rendering
    ModelSelector.tsx       # LLM model dropdown
    Leaderboard.tsx         # Benchmark leaderboard
    CommentaryPanel.tsx     # Live commentary display
    LiveStats.tsx           # In-game statistics
  commentary/     # AI commentary system
    commentaryQueue.ts  # Move batching and commentary generation queue
    batchPrompt.ts      # Commentary prompt construction
    fillerPrompts.ts    # Idle-time filler commentary
  chess/          # Chess engine wrappers
    stockfish.ts    # Stockfish Web Worker wrapper (singleton)
    board.ts        # Chess.js wrapper with game-over detection
  tts/            # Text-to-speech
    audio-queue.ts  # Audio playback queue
    tts-client.ts   # TTS synthesis client (local + cloud)
  benchmark/      # Benchmark data pipeline
    aggregates.ts   # Statistical aggregation (Elo, win rates, CPL)
    mapper.ts       # Game -> benchmark record mapping
    profile.ts      # Player config fingerprinting
    db.ts           # IndexedDB wrapper
    repository.ts   # Benchmark data persistence
    reports.ts      # Benchmark report generation
    export.ts       # ZIP export for benchmark data
    types.ts        # Benchmark type definitions
    legacy.ts       # Legacy data migration
  pgn/            # PGN handling
    parser.ts       # PGN file parser
    replay-runtime.ts   # PGN replay engine
  tts/            # Text-to-speech
  utils/          # Shared utilities
    export.ts       # PGN/CSV/JSON export (~898 lines)
    metrics.ts      # Game quality metrics computation
    chess-squares.ts # Square highlighting utilities
    board-annotations.ts # Board arrow/highlight annotations
    elo.ts          # Elo rating calculation
  App.tsx         # Main app with tab routing (Game/Tournament/Replay)
  main.tsx        # Entry point
```

### Key Patterns

- **Game Runtime:** `runtime.ts` is the central game loop. It handles 5 player types (stockfish, oracle, replay, human, LLM) in a single `gameLoop()` method. Emits `GameEvent`s processed by `reducer.ts`.
- **State Flow:** Runtime emits events -> reducer produces new GameState -> store subscribers update UI.
- **Event Log:** All game events are appended to `GameState.eventLog` (immutable array spreads).
- **LLM Clients:** Factory pattern via `createLLMClient()`. Each client implements `callChat()`, `requestCommentary()`, `requestCommentaryStream()`, `listModels()`.
- **Tournament Store:** Largest store (~1474 lines). Manages matches, pairs, games, auto-play, and persistence via `syncToMap`.
- **Module Singletons:** Stockfish instances, commentary queue callbacks, and various counters are module-level `let` variables. Fragile during HMR.

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server (Vite)
npm run build        # Production build
npm run lint         # ESLint
npm run preview      # Preview production build
```

## Known Issues (from CODE_REVIEW.md)

### Critical Bugs to Be Aware Of
1. **runtime.ts:480** — LLM retry loop never enforces `maxRetries`. Infinite retries on persistent illegal moves.
2. **runtime.ts:264+** — `isCheckmate` hardcoded to `false` in all MoveApplied events.
3. **oracle-player.ts:68** — SAN vs UCI notation comparison never matches. Oracle correction always enters fallback.
4. **response-toolkit.ts:131** — `processCompromisedDeclaration` hardcodes White as winner.
5. **openai.ts:189, openrouter.ts:179** — Checks `onToken` instead of `isStreaming`, causing JSON-as-SSE parsing.
6. **benchmarkStore.ts:175-213** — Race condition in `recordGame` loses data on concurrent calls.
7. **mapper.ts:206** — Centipawn loss computed incorrectly (absolute eval, not delta from best move).
8. **stockfish.ts:374** — `formatEval` mate display has identical ternary branches (copy-paste bug).

### Systemic Issues
- **~500 lines of code duplication** across LLM clients (readStream, callWithRetry, commentary) and components (GameControls/TournamentSetup share PlayerSlotEditor, types, constants).
- **O(n^2) event log growth** from `[...state.eventLog, event]` on every event in the reducer.
- **API keys in plaintext localStorage** with no encryption or expiration.
- **Non-deterministic game logic** (`Math.random()` without seed) makes benchmarks non-reproducible.
- **Module-level mutable state** (singletons, counters) breaks during HMR and complicates testing.

### Performance Hotspots
- Tournament state: deep spreading (3-4 levels) on every game event.
- Multiple unmemoized derived computations in components (Leaderboard, TournamentProgress, LiveStats).
- Stockfish eval handler duplicated in 3 places.
- `utils/export.ts:542` — O(n^2) game lookup in tournament JSON export.

## Development Notes

- No test suite exists. Manual testing only.
- ESLint is configured but several `eslint-disable` comments mask real dependency issues (especially in effects).
- The `tournamentStore.ts` file is the most complex and bug-prone. Handle with care.
- Stockfish runs in Web Workers. The singleton pattern means two instances max (eval + player).
- The commentary system uses a queue with batching, filler generation, and optional TTS narration gating.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mnehmos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mnehmos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
