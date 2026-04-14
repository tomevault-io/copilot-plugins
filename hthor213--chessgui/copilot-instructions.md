## chessgui

> A macOS-first chess GUI built on Tauri 2 (Rust) + React + TypeScript. Uses Lichess Chessground for the board and chessops for move logic. Designed to replace ChessBase with an open-source, subscription-free alternative powered by Stockfish.

# ChessGUI — Claude Code Project Context

## What This Is
A macOS-first chess GUI built on Tauri 2 (Rust) + React + TypeScript. Uses Lichess Chessground for the board and chessops for move logic. Designed to replace ChessBase with an open-source, subscription-free alternative powered by Stockfish.

## Architecture
- **Frontend:** Next.js (static export) + React 19 + TypeScript + Tailwind CSS + shadcn/ui + Chessground board
- **Backend:** Rust via Tauri v2 — handles UCI engine communication over stdin/stdout
- **Build:** Next.js + pnpm + Cargo
- **Target:** macOS (Apple Silicon primary), cross-platform possible via Tauri

## Development Commands
```bash
# Dev mode (hot-reload frontend + Rust backend)
source "$HOME/.cargo/env" && pnpm tauri dev

# Build debug .app + .dmg
source "$HOME/.cargo/env" && pnpm tauri build --debug

# Build release
source "$HOME/.cargo/env" && pnpm tauri build

# Frontend only (no Tauri shell)
pnpm dev

# Type check
pnpm tsc --noEmit
```

## Project Structure
```
app/                    # Next.js app directory
  layout.tsx            # Root layout (dark theme)
  page.tsx              # Main chess UI ("use client", three-column grid)
  globals.css           # Tailwind imports + Chessground overrides
components/             # React components
  board.tsx             # Chessground wrapper (dynamic import, SSR-safe)
  move-list.tsx         # Move list (shadcn Card + ScrollArea)
  analysis-panel.tsx    # Engine eval display (shadcn Card + Badge + Button)
  pgn-import-dialog.tsx # PGN import (shadcn Dialog + Textarea)
  promotion-dialog.tsx  # Piece picker overlay
  ui/                   # shadcn/ui generated components
hooks/                  # React hooks
  use-chess-game.ts     # Game state, legal moves via chessops
  use-engine.ts         # Stockfish UCI communication via Tauri
lib/                    # Utility libraries
  uci-parser.ts         # UCI info line parser
  utils.ts              # cn() helper (shadcn)
src-tauri/              # Rust backend
  src/
    lib.rs              # Tauri plugin setup + command registration
    main.rs             # Entry point
    uci.rs              # UCI engine process management
  tauri.conf.json       # App config, window, bundle settings
  Cargo.toml            # Rust dependencies
specs/                  # Feature specs (vision + MVP)
scripts/                # Utility scripts
```

## Key Dependencies
- `@lichess-org/chessground` — Board rendering (GPL-3.0)
- `chessops` — Move generation, FEN/PGN, validation
- `next` — React framework (static export for Tauri)
- `tailwindcss` — Utility-first CSS
- `shadcn/ui` — Composable UI components (Radix primitives)
- `tauri` + `tauri-plugin-shell` — Desktop shell + engine process management
- `tokio` — Async runtime for engine I/O in Rust

## Spec System
Uses band-numbered specs in `specs/`. See `specs/README.md` for the full index and dependency graph.

Active specs:
- 000 (vision), 001 (board & gameplay), 002 (UX/UI migration)
- 011 (engine analysis), 013 (PGN import/export), 016 (game tree)
- 200 (database & opening explorer), 202 (annotations & eval graph)
- 900 (backlog)

## License
GPL-3.0 (required by Chessground dependency)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hthor213)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hthor213)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
