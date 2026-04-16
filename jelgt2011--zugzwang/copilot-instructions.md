## zugzwang

> A Next.js chess puzzle training application using puzzles from the Lichess database. Users can practice tactics, track their progress, and improve their chess skills.

# Zugswang - Chess Puzzle Training App

## Project Overview
A Next.js chess puzzle training application using puzzles from the Lichess database. Users can practice tactics, track their progress, and improve their chess skills.

## Tech Stack
- **Framework**: Next.js 16 with App Router and Turbopack
- **Language**: TypeScript (strict)
- **Auth & Database**: Firebase (Auth + Firestore)
- **State Management**: Zustand (stores in `src/stores/`)
- **UI**: Tailwind CSS v4 + shadcn/ui components (Radix primitives)
- **Chess**: chess.js for logic, react-chessboard for rendering, Stockfish for analysis
- **AI**: OpenAI Agents SDK for hints/coaching

## Project Structure
```
src/
├── app/           # Next.js App Router pages and API routes
├── components/    # React components (ui/ for shadcn primitives)
├── contexts/      # React contexts (Auth, Stockfish)
├── hooks/         # Custom React hooks
├── lib/           # Utility functions and services
├── stores/        # Zustand stores
└── types/         # TypeScript type definitions
```

## General Rules

### CLI First
- Prefer CLI tools when possible (e.g., use `mv` to rename a file, then update references)
- Use shell commands for file operations when it's simpler than editor operations

### Incremental Progress
- Make progress incrementally - ensure no errors exist in a file before moving on
- Run linting/type checking after significant changes
- Test changes in isolation when possible

### Code Style
- Use TypeScript strictly - avoid `any` types
- Prefer named exports over default exports
- Keep components focused and composable
- Use Zustand stores for shared state, React state for local UI state

## Conventions

### Stores (Zustand)
- Use `devtools` middleware for debugging
- Action names should be descriptive (e.g., `makeMove:correct`, `startPuzzle`)
- Keep computed values as getter functions (e.g., `getFilteredPuzzles`)

### Components
- Use shadcn/ui components from `@/components/ui/` for consistency
- Chess-related components: `ChessGame.tsx`, `PuzzleBoard.tsx`
- Keep page components in `app/` thin - extract logic to hooks

### Types
- Puzzle types in `src/types/puzzle.ts`
- User types in `src/types/user.ts`
- Use Zod for runtime validation where needed

### API Routes
- Located in `src/app/api/`
- Use Firebase Admin SDK for server-side operations

## Development
- Dev server runs on port 3001: `npm run dev`
- Lint with: `npm run lint`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JELGT2011) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
