## trustaasldb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SvelteKit 5 application with SQLite database, Drizzle ORM, and Better Auth authentication. Uses Svelte 5 with runes.

## Common Commands

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run preview      # Preview production build
npm run check        # Type check with svelte-check
npm run lint         # Run Prettier and ESLint
npm run format       # Format code with Prettier
npm run db:push      # Push Drizzle schema to database
npm run db:generate  # Generate Drizzle migrations
npm run db:migrate   # Run Drizzle migrations
npm run db:studio    # Open Drizzle Studio
npm run auth:schema  # Generate better-auth schema
npm run storybook    # Run Storybook on port 6006
```

## Architecture

- **Database**: SQLite via libsql client with Drizzle ORM
- **Authentication**: Better Auth with SvelteKit integration
- **Styling**: Tailwind CSS 4 with forms plugin
- **Testing**: Vitest + Playwright for browser tests

### Key Files

- `src/lib/server/auth.ts` - Better Auth configuration
- `src/lib/server/db/schema.ts` - Database schema (custom `task` table + auth tables)
- `src/lib/server/db/index.ts` - Drizzle database client
- `src/hooks.server.ts` - Server hooks for authentication session handling

### Routes

- `/` - Home page
- `/demo` - Demo page
- `/demo/better-auth` - Authentication demo with login page

### Environment Variables

Required in `.env`:
- `DATABASE_URL` - SQLite database path (e.g., `file:local.db`)
- `ORIGIN` - Application origin URL
- `BETTER_AUTH_SECRET` - Secret for Better Auth (32+ chars for production)

---
> Source: [bluecatgreen/trustAaSLDB](https://github.com/bluecatgreen/trustAaSLDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
