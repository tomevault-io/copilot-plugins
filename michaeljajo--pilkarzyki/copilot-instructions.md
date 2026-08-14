## pilkarzyki

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Always refer to @RULES.md and apply rules which are listed there.

## Development Commands

### ✨ V2 System - Bulletproof Dev Server Management

- **Development server**: `npm run dev` (uses Turbopack for faster builds)
  - **V2 Features**: Process Manager, orphan detection, atomic locking
  - Zero manual intervention required
  - Automatic comprehensive cleanup when needed
  - See `DEV_SERVER_GUIDE.md` for full V2 documentation

- **Monitored mode**: `npm run dev:monitored` (auto-healing with V2 health monitor)
  - Memory leak detection (auto-restart on 30% growth)
  - CPU spike detection (auto-restart on sustained high CPU)
  - Multi-endpoint health checks
  - Perfect for long sessions (3+ hours)

- **Status dashboard**: `npm run dev:status` (real-time process visibility)
  - Server health and metrics
  - All registered/orphaned processes
  - Ports in use
  - Performance graphs

- **Watch mode**: `npm run dev:watch` (live updating dashboard)

### Cleanup Commands V2

- **Comprehensive cleanup**: `npm run dev:cleanup` (uses ProcessManager)
  - Kills orphaned processes
  - Frees all ports
  - Shows detailed summary

- **Emergency cleanup**: `npm run dev:emergency` (nuclear option)
  - Works when everything else fails
  - Survives ProcessManager corruption
  - Guaranteed to work

- **Force restart**: `npm run dev:force` (emergency cleanup + start)
- **Legacy cleanup**: `npm run kill:all` (aggressive cleanup script)
- **Full clean**: `npm run clean` (emergency + cache removal)
- **Quick port kill**: `npm run kill:dev` (port 3000 only)

### Monitoring Commands

- **View logs**: `npm run dev:logs` (tail health monitor logs)
- **Safe mode**: `npm run dev:safe` (graceful shutdown on Ctrl+C)

### Build & Deploy

- **Build**: `npm run build` (production build with Turbopack)
- **Start**: `npm start` (production server)
- **Lint**: `npm run lint` (ESLint)

### Important Notes

- **Server Stability V2**: Localhost:3000 issues PERMANENTLY RESOLVED with 5-layer defense
- **Process Manager**: Centralized PID registry tracks all processes with parent-child relationships
- **Orphan Detection**: Finds and kills processes whose parents died (PPID verification)
- **Atomic Locking**: Prevents concurrent cleanup race conditions
- **Memory Leak Detection**: Auto-restart on 30% memory growth over 10 samples
- **CPU Spike Detection**: Auto-restart on >80% CPU for 3 consecutive checks
- **System Watchdog**: Optional macOS launchd integration for boot cleanup
- **Zero Intervention**: System self-heals from all failure modes
- **Next.js 16 Migration**: This project uses `proxy.ts` (not `middleware.ts`) for route handling
- **Troubleshooting**: See `DEV_SERVER_GUIDE.md` for complete V2 guide

## Architecture Overview

This is a fantasy football management application built with Next.js 15, featuring:

### Authentication & Authorization
- **Clerk** for user authentication with middleware protection
- Routes are protected by default except: `/`, `/sign-in/*`, `/sign-up/*`, `/api/webhooks/*`
- Middleware at root level handles auth routing

### Database Architecture
- **Supabase** PostgreSQL backend with three client configurations:
  - `src/lib/supabase.ts`: Basic client and admin client
  - `src/lib/supabase-client.ts`: Client-side with Clerk auth integration
  - `src/lib/supabase-server.ts`: Server-side with Clerk auth integration
- Clerk tokens are passed as Authorization headers to Supabase for RLS

### Core Data Models
Key entities defined in `src/types/index.ts`:
- **User**: Clerk ID integration with admin roles
- **League**: Season management with gameweek tracking
- **Player**: Squad members with position/manager assignment
- **Squad**: Manager's player collections
- **Lineup**: Weekly team selections (3 players max)
- **Gameweek**: Time-based periods with lock dates
- **Match**: Head-to-head fixtures between managers
- **Result**: Player goal scoring data

### Application Structure
- **App Router** with TypeScript
- **Protected routes** under `/dashboard/*`
- **Clerk auth pages** at `/sign-in` and `/sign-up`
- **Root layout** wraps app in ClerkProvider
- **Tailwind CSS** for styling with Geist fonts

### Key Business Logic
- **Squad Management**: Excel import for player assignment to managers
- **Lineup Validation**: 3-player limit per gameweek with position rules
- **Scoring System**: Goal-based with automated league standings
- **Scheduling**: Double round-robin for up to 16 managers
- **Real-time Updates**: Live sync via Supabase subscriptions

### Deployment Configuration
- **Vercel** optimized with `vercel.json`
- **Regional deployment**: Frankfurt (fra1)
- **Function timeout**: 30 seconds for API routes
- **Turbopack** for faster development and builds

## Environment Variables Required

```bash
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_SIGN_IN_FORCE_REDIRECT_URL=/dashboard
NEXT_PUBLIC_CLERK_SIGN_UP_FORCE_REDIRECT_URL=/dashboard

# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
DATABASE_URL=
```

## Development Notes

- Use appropriate Supabase client based on context (client/server/admin)
- All database operations should respect Clerk user context
- TypeScript interfaces are comprehensive - reference `src/types/index.ts`
- Authentication is handled at middleware level, not component level
- Position types are strictly enforced: 'Goalkeeper', 'Defender', 'Midfielder', 'Forward'

---
> Source: [michaeljajo/Pilkarzyki](https://github.com/michaeljajo/Pilkarzyki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
