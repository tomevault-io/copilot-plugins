## 1up

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**1up** is a productivity time tracking application built with Next.js, TypeScript, and PostgreSQL. It runs on Vercel
and uses Supabase as its postgres provider.

## Tech Stack

- **Frontend**: Next.js 15.3.1, React 19, TypeScript, Tailwind CSS 4.0
- **Database**: PostgreSQL with Drizzle ORM via Supabase
- **Infrastructure**: Vercel
- **Package Management**: pnpm

## Development Commands

### Core Commands (run from project root)

```bash
pnpm install          # Install dependencies
pnpm run dev          # Start Next.js development server
pnpm run build        # Production build
pnpm run start        # Start production server
pnpm run test         # Run Jest test suite
pnpm run lint         # ESLint checking
pnpm run lint:fix     # Auto-fix linting issues
pnpm run prettier     # Format all code with Prettier
```

### Database Commands (Drizzle)

```bash
pnpm db:generate            # Generate migrations
pnpm db:migrate:staging     # Run migrations against staging
pnpm db:migrate:prod        # Run migrations against prod
```

## Architecture

### Project Structure

This is a full-stack Next.js application with the following key directories:

- **`src/app/`**: Next.js App Router pages and layouts
- **`src/app/components/`**: Reusable React components
- **`src/app/(routes)/`**: Route groups for organized routing
- **`src/models/`**: TypeScript interfaces and types
- **`src/server/`**: Server-side code including database schemas and queries
- **`src/server/db/`**: Drizzle ORM schemas and database configuration
- **`db/scripts/`**: SQL initialization scripts
- **`public/images/`**: Static image assets

### Database Layer

- **ORM**: Drizzle ORM for type-safe database operations
- **Schema**: Located in `src/server/db/schemas.ts`
- **Configuration**: `drizzle.config.ts` handles database connection
- **Queries**: Server actions in `src/server/queries.ts`

### Frontend Structure

- **App Router**: Uses Next.js 15+ app directory structure
- **Server Actions**: Database operations handled through Next.js server actions
- **Components**: Modular React components with Tailwind CSS
- **Routes**: Timer details at `/timers/[id]` with dynamic routing
- **Theme Support**: Dark/light mode with next-themes

## Environment Setup

Environment variables are set and stored in the Vercel dashboard keyed by deployment environment. For local
development environment variables are pulled in from vercel using `vercel env pull` and put in a `.env.local` file.

Notably, local development points towards a staging DB rather then production.

## Testing

- **Framework**: Jest with React Testing Library
- **Configuration**: `jest.config.ts` and `jest.setup.ts`
- **Test Location**: Tests should be placed alongside components or in `__tests__` directories

## Code Style

- **Imports**: Use absolute imports (e.g. `@/lib/utils`) unless the import is a sibling of the importing file, in which case use a relative import (e.g. `./use-login-form`).

## Code Quality

### CI/CD

GitHub Actions workflows enforce:

- Code formatting (`.github/workflows/formatting.yaml`)
- Linting (`.github/workflows/linting.yaml`)
- Testing (`.github/workflows/testing.yaml`)

### Tools

- **Linting**: ESLint with TypeScript support
- **Formatting**: Prettier with Tailwind CSS plugin
- **Type Checking**: TypeScript with strict configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dvochoa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
