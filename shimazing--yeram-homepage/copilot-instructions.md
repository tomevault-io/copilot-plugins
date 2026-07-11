## yeram-homepage

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
```bash
npm run dev          # Start development server with hot reload (port 5173)
npm run build        # Build for production (client + server)
npm start            # Run production build
npm run check        # Run TypeScript type checking
```

### Database Management
```bash
npm run db:push      # Push schema changes to PostgreSQL database
npm run db:migrate   # Migrate data from data.json to PostgreSQL (one-time)
```

**Important**:
- Requires `DATABASE_URL` environment variable in `.env` file
- `.env.example` provided as template
- Currently using Railway PostgreSQL in production

## Architecture Overview

### Monorepo Structure
This is a full-stack TypeScript monorepo with three main directories:
- **`client/`**: React SPA (Vite + React + TanStack Query + Wouter routing)
- **`server/`**: Express API server
- **`shared/`**: Shared TypeScript code (schemas, types)

### Single Server, Single Port
- **Port 5173** serves both the API and the React client
- In development: Vite dev server handles client, Express serves `/api/*` routes
- In production: Express serves static built files from `dist/public/` and API routes
- This architecture is required for deployment environments with port restrictions

### Build Process
The build creates two artifacts:
1. **Client build** (Vite): React app bundled to `dist/public/`
2. **Server build** (esbuild): Express server bundled to `dist/index.js`

### Data Layer Architecture

**Current State**: Uses **PostgreSQL database** via Drizzle ORM (`DrizzleStorage` in `server/storage.ts`)
- Data persists across server restarts
- Automatic fallback to `FileStorage` (data.json) if `DATABASE_URL` not set
- All storage methods are async and implement the `IStorage` interface

**Database Schema**: PostgreSQL schema defined in `shared/schema.ts` using Drizzle ORM
- Tables: `users`, `announcements`, `galleryPhotos`, `weeklyService`
- Zod validation schemas derived from Drizzle schema
- Hosted on Railway PostgreSQL

**Storage Classes**:
- `DrizzleStorage`: PostgreSQL storage (active when DATABASE_URL is set)
- `FileStorage`: JSON file storage (fallback, saves to data.json)
- `MemStorage`: In-memory storage (legacy, data lost on restart)

### API Structure
All API routes in `server/routes.ts` follow this pattern:
1. Parse/validate request data using Zod schemas from `@shared/schema`
2. Call storage interface methods
3. Return JSON responses with error handling

Available endpoints:
- `GET /api/announcements/current` - Get latest announcement
- `PUT /api/announcements` - Update announcement
- `GET /api/gallery?category=worship|fellowship|youth|events` - Get gallery photos
- `POST /api/gallery` - Add photo
- `DELETE /api/gallery/:id` - Delete photo
- `GET /api/weekly-service` - Get current week's service info
- `PUT /api/weekly-service` - Update weekly service

### Frontend Architecture
- **Routing**: Wouter (lightweight client-side routing)
- **State Management**: TanStack Query for server state
- **UI Components**: Radix UI primitives + custom components in `client/src/components/ui/`
- **Styling**: Tailwind CSS with custom theme configuration
- **Pages**: Home, Services, Announcements, Gallery (all in `client/src/pages/`)

### Path Aliases
Both build tools and TypeScript recognize these aliases:
- `@/*` → `client/src/*`
- `@shared/*` → `shared/*`
- `@assets/*` → `attached_assets/*`

## Important Notes

### Environment Configuration
- Development mode automatically detected via `NODE_ENV=development`
- Vite dev middleware only loaded in development (see `server/index.ts:53-56`)
- Production serves pre-built static files
- **Required environment variables** (in `.env` file):
  - `DATABASE_URL`: PostgreSQL connection string (Railway PostgreSQL)
  - `NODE_ENV`: `development` or `production`

### Type Safety
- Shared types ensure client/server contract consistency
- All API request/response bodies typed via `@shared/schema`
- Run `npm run check` before committing to catch type errors

### Drizzle ORM Integration
Schema file at `shared/schema.ts` defines:
- Table structures with `pgTable`
- Insert schemas with validation (using `createInsertSchema`)
- TypeScript types inferred from schemas

**Database Setup** (already configured):
1. Railway PostgreSQL database connected
2. Schema synced via `npm run db:push`
3. Data migrated from data.json via `npm run db:migrate`
4. `DrizzleStorage` automatically used when `DATABASE_URL` is present

**To switch databases**:
1. Update `DATABASE_URL` in `.env` file
2. Run `npm run db:push` to create tables
3. Run `npm run db:migrate` to import existing data (if needed)

## Deployment

### Production Environment
- **Hosting Platform**: Railway (https://railway.app)
- **Domain**: www.yeram.me.kr
- **Domain Provider**: 호스팅케이알 (hosting.kr)
- **Deployment**: Automatic deployment on push to `main` branch

### Railway Configuration

**Environment Variables (set in Railway dashboard)**:
- `VITE_SITE_URL`: https://www.yeram.me.kr (used for SEO meta tags)
- `UPLOAD_DIR`: /data/uploads (persistent storage for gallery images)
- `DATABASE_URL`: PostgreSQL connection string (Railway PostgreSQL)
- `NODE_ENV`: production

**Railway Volume**:
- Mount path: `/data`
- Purpose: Persistent storage for uploaded gallery images
- Images stored in: `/data/uploads`
- **Important**: Without Volume, uploaded images are lost on redeployment

### File Storage
- **Gallery Images**: Stored in `/data/uploads` (Railway Volume)
- **Local Development**: Images stored in `./attached_assets`
- **Static Assets**: Served via `/@assets` route (see `server/index.ts`)
- Images persist across redeployments when using Railway Volume

### SEO Configuration
- `robots.txt` and `sitemap.xml` are dynamically generated by server (see `server/routes.ts`)
- Meta tags in `client/index.html` use `%VITE_SITE_URL%` placeholder
- Registered with Google Search Console and Naver Search Advisor

---
> Source: [shimazing/Yeram-Homepage](https://github.com/shimazing/Yeram-Homepage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
