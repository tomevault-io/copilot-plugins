## signallost

> Signal Lost - A Lethal Company-like 3D multiplayer web game.

# Claude Instructions

## Project Overview
Signal Lost - A Lethal Company-like 3D multiplayer web game.

## Before Starting Any Task
1. Read `TODO.md` to understand current progress
2. Identify which phase/task you're working on
3. Mark task as in-progress in TODO.md

## After Completing Any Task
1. Update `TODO.md` - mark completed tasks with `[x]`
2. Commit changes with clear message
3. Push to branch

## Commands
```bash
# Development
pnpm install          # Install dependencies
pnpm dev              # Start dev servers (web + game server)

# Production
pnpm build            # Build for production
pnpm start            # Start production servers

# Individual
pnpm dev:web          # Web client only (port 3000)
pnpm dev:server       # Game server only (port 2567)
```

## Code Principles
- **Minimal code** - Write the least code necessary
- **No over-engineering** - Solve the current problem, not hypothetical ones
- **No premature abstraction** - Duplicate code is fine until patterns emerge
- **Clean and readable** - Simple > clever
- **No unnecessary comments** - Code should be self-documenting
- **No unused code** - Delete it, don't comment it out

## Tech Stack
- Frontend: Next.js 14 + React Three Fiber + Rapier
- Backend: Colyseus game server
- Voice: Simple-peer (WebRTC)
- State: Zustand

## Structure
```
apps/web/        # Next.js client
apps/server/     # Colyseus server
packages/shared/ # Shared types
```

---
> Source: [Fowthy/signallost](https://github.com/Fowthy/signallost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
