## typio-kart

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Typio-Kart** is a multiplayer typing racing game similar to TypeRacer, but with Mario Kart-inspired powerup mechanics that allow players to sabotage other racers. The project is built on the T3 Stack.

### Development Roadmap
1. **Phase 1**: Basic typing game mechanics (single player)
   - Text rendering and cursor tracking
   - WPM/accuracy calculations
   - Basic race progress visualization
2. **Phase 2**: Multiplayer functionality
   - Real-time connections between racers
   - Live race state synchronization
   - Lobby and matchmaking system
3. **Phase 3**: Powerup system
   - Collectible powerups during races
   - Sabotage mechanics (text scrambling, cursor interference, etc.)
   - Defensive/offensive powerup balance

## Essential Commands

### Development
- `pnpm dev` - Start Next.js development server with Turbo
- `pnpm build` - Build production bundle
- `pnpm start` - Start production server
- `pnpm preview` - Build and start production server

### Code Quality
- `pnpm check` - Run both linting and type checking
- `pnpm lint` - Run ESLint
- `pnpm lint:fix` - Run ESLint with auto-fix
- `pnpm typecheck` - Run TypeScript type checking (tsc --noEmit)
- `pnpm format:check` - Check code formatting with Prettier
- `pnpm format:write` - Format code with Prettier

### Database (Prisma)
- `pnpm db:generate` - Generate Prisma client and run migrations (dev)
- `pnpm db:migrate` - Deploy migrations (production)
- `pnpm db:push` - Push schema changes directly to database
- `pnpm db:studio` - Open Prisma Studio database browser

## Architecture Overview

This is a **T3 Stack** application using:
- **Next.js 15** with App Router
- **tRPC** for type-safe API endpoints
- **Prisma** with PostgreSQL for database
- **NextAuth.js v5** for authentication
- **Tailwind CSS v4** for styling
- **Zod** for runtime type validation

### Key Architectural Patterns

**tRPC Setup**: Server procedures are defined in `src/server/api/routers/` and exported through `src/server/api/root.ts`. Use `publicProcedure` for unauthenticated endpoints and `protectedProcedure` for authenticated ones.

**Authentication Flow**: NextAuth.js configuration in `src/server/auth/config.ts` with Discord provider. Session context is available in tRPC procedures through `ctx.session`.

**Database Layer**: Prisma schema defines User/Post models with NextAuth.js adapter tables. The `db` instance is available globally through `src/server/db.ts`.

**Environment Variables**: Managed through `@t3-oss/env-nextjs` in `src/env.js` with Zod validation. Required vars: `DATABASE_URL`, `AUTH_DISCORD_ID`, `AUTH_DISCORD_SECRET`, and `AUTH_SECRET` (production only).

**Client-Server Communication**: tRPC client configured in `src/trpc/` with React Query integration. Server-side calls use `src/trpc/server.ts`.

### Game-Specific Architecture Considerations

**Real-time Features**: For Phase 2, consider WebSocket integration for live race synchronization. tRPC supports subscriptions, but WebSockets may be needed for low-latency powerup interactions.

**Game State Management**: Typing game state (current position, WPM, accuracy) should be managed client-side with periodic server sync. Race progress and powerup effects need server authority to prevent cheating.

**Performance**: Typing games require responsive input handling. Consider React's concurrent features and debouncing for WPM calculations.

### File Structure Conventions
- API routes: `src/server/api/routers/`
- React components: `src/app/_components/`
- Game components: `src/app/_components/game/` (recommended)
- Path alias `@/*` maps to `src/*`
- Database models follow NextAuth.js adapter schema requirements

## Development Setup
1. Install dependencies: `pnpm install`
2. Set up environment variables (copy `.env.example`)
3. Set up database: `pnpm db:push`
4. Start development: `pnpm dev`

## Game Development Notes

### Typing Game Mechanics
- Track keystroke accuracy and timing
- Calculate real-time WPM (Words Per Minute)
- Handle common typing scenarios (backspace, corrections)
- Visual progress indicators (racing car position)

### Future Database Schema Considerations
- Race sessions and results
- User statistics and leaderboards
- Powerup definitions and effects
- Real-time race state (may need Redis/in-memory store)

---
> Source: [vinnyhuang-devin-test/typio-kart](https://github.com/vinnyhuang-devin-test/typio-kart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
