## ashtonvalephoto

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Abbie Street Photo & Video - A luxury wedding photography/videography marketing and inquiry platform built with React, Express, and PostgreSQL.

## Commands

```bash
npm run dev            # Full-stack development (Express + Vite HMR)
npm run dev:client     # Frontend only (Vite dev server)
npm run build          # Production build (client + server)
npm start              # Run production server
npm run check          # TypeScript type checking
npm run db:push        # Push Drizzle schema changes to database
```

## Architecture

### Directory Structure
- `client/` - React 19 frontend with Vite
- `server/` - Express backend with TypeScript
- `shared/` - Shared code (Drizzle schema, Zod validation)
- `db/` - Database initialization (Neon PostgreSQL)
- `script/` - Build scripts (esbuild config)

### Tech Stack
- **Frontend**: React 19, Vite 7, Wouter (routing), TanStack Query, shadcn/ui, Tailwind CSS v4, Framer Motion
- **Backend**: Express, Drizzle ORM, Zod validation
- **Database**: PostgreSQL via Neon Serverless
- **Build**: Vite (client), esbuild (server)

### Key Patterns

**Path Aliases** (configured in tsconfig.json and vite.config.ts):
- `@/*` → `client/src/*`
- `@shared/*` → `shared/*`
- `@/db` → `db/index.ts`

**Routing**: Uses Wouter (not React Router) - lightweight client-side router

**State Management**:
- Server state via TanStack Query with custom queryClient (`client/src/lib/queryClient.ts`)
- Form state via React Hook Form with Zod resolvers

**Validation**: Zod schemas in `shared/schema.ts` are used by both client forms and server API validation

**Storage Layer**: `server/storage.ts` implements IStorage interface for database abstraction

### API Endpoints (all prefixed with `/api`)

| Endpoint | Methods | Purpose |
|----------|---------|---------|
| `/inquiries` | GET, POST | Wedding inquiry management |
| `/inquiries/:id` | GET | Single inquiry |
| `/inquiries/:id/status` | PATCH | Update inquiry status |
| `/photographers` | GET | All photographers |
| `/photographers/featured` | GET | Featured photographers |
| `/portfolio` | GET | Portfolio items (supports `?category=` filter) |
| `/portfolio/featured` | GET | Featured portfolio items |
| `/reviews` | GET, POST | Customer reviews |
| `/reviews/featured` | GET | Featured reviews |

### Database Schema (shared/schema.ts)

Core tables: `users`, `inquiries`, `photographers`, `portfolioItems`, `reviews`

### UI Components

- `/client/src/components/ui/` - 30+ shadcn/ui components (New York style)
- `/client/src/components/` - Feature components (Hero, Navigation, Footer, etc.)
- `/client/src/pages/` - Page components matching routes

### Environment Variables

- `DATABASE_URL` - PostgreSQL connection string (required)
- `PORT` - Server port (default 5000)
- `NODE_ENV` - "development" or "production"

## Business Context

This is a luxury wedding photography brand with tiered pricing packages (Bespoke, Estate, Signature, Editorial). The style quiz feature matches couples to appropriate packages based on their aesthetic preferences.

---
> Source: [thephotocrm/ashtonvalephoto](https://github.com/thephotocrm/ashtonvalephoto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
