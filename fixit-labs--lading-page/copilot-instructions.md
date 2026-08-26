## lading-page

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ParKpool landing page - a Next.js 14 App Router application for a valet parking SaaS platform. The landing page captures leads through a demo request modal and uses Prisma with SQLite for data persistence.

**Key Tech Stack:**
- Next.js 14 (App Router)
- TypeScript with strict mode
- Tailwind CSS + Framer Motion
- Prisma ORM (SQLite, migrating to PostgreSQL)
- Zod for validation
- Custom i18n implementation (es/en)

**Brand Colors:**
- Azure Blue: #0A62F8
- Midnight Navy: #0B2848

## Development Commands

```bash
# Install dependencies (uses pnpm)
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Lint code
npm run lint

# Database operations
cd prisma
npx prisma db push        # Push schema changes to database
npx prisma generate       # Generate Prisma client
npx prisma studio         # Open database GUI
```

**Package Manager:** Project uses pnpm@9.9.0 (specified in package.json). npm commands work but pnpm is preferred.

## Architecture

### Client/Server Boundaries

This app uses Next.js App Router with clear client/server separation:

- **Server Components** (default): `app/layout.tsx` handles metadata and font loading
- **Client Components** ('use client'): All interactive components including `app/page.tsx`, `components/LandingPage.tsx`, `components/DemoRequestModal.tsx`, and the entire Provider tree

The root layout wraps children in `<Providers>` component which sets up the `LocaleProvider` for i18n state management.

### Internationalization Architecture

Custom i18n implementation (not using next-intl despite it being in dependencies):

1. **Context-based:** `contexts/LocaleContext.tsx` provides `useLocale()` hook
2. **Translation files:** `messages/es.json` and `messages/en.json` contain all UI strings
3. **Provider setup:** `components/Providers.tsx` wraps app with `LocaleProvider`
4. **Current state:** Defaults to Spanish ('es'), English support prepared but not fully integrated

Components access translations via `useLocale()` hook and load appropriate message file.

### Database Layer

**Prisma Setup:**
- SQLite database at `prisma/dev.db` (development)
- Prisma client singleton in `lib/prisma.ts` prevents hot-reload connection issues
- Schema uses snake_case database columns (`@map`) with camelCase TypeScript fields

**Lead Model:**
- Unique email constraint
- Status field: "NEW", "CONTACTED", "QUALIFIED", "CONVERTED", "REJECTED" (string-based, SQLite limitation)
- UUID primary keys

**Important:** The API endpoint (`app/api/v1/public/leads/route.ts`) currently logs leads but doesn't save to database (TODO comment present). The Prisma client is set up but not yet integrated.

### API Structure

Public API endpoints follow `/api/v1/public/*` pattern:
- `POST /api/v1/public/leads` - Lead creation endpoint
- Uses Zod schema validation before processing
- Returns Spanish error messages (hardcoded)

## Path Aliases

Uses `@/*` to reference project root:
```typescript
import { prisma } from '@/lib/prisma';
import LandingPage from '@/components/LandingPage';
```

## Styling Approach

- **Tailwind CSS** for all styling (no CSS modules or styled-components)
- **Framer Motion** for animations
- **Google Fonts:** Noto Sans (400, 500, 700, 900) loaded in layout
- **Responsive design** using Tailwind breakpoints
- **Lucide React** for icons

## Important Implementation Notes

1. **Database Not Connected:** The lead form submits successfully but only logs data (line 20 in `app/api/v1/public/leads/route.ts`). Prisma integration is TODO.

2. **Migration Path:** Planned migration from SQLite to PostgreSQL. When implementing, update `prisma/schema.prisma` datasource and consider enum types for Lead.status.

3. **i18n Incomplete:** Translation infrastructure exists but language switcher may not be fully functional. The `LanguageSwitcher` component exists but English content may be incomplete.

4. **Terms Page:** Located at `/app/terms/page.tsx` - linked from demo modal.

5. **Client-Side State:** All interactive state lives in client components. No global state management beyond Context (no Redux/Zustand).

## Git Status Note

Multiple modified files are uncommitted. When making changes, be aware that the working directory has ongoing modifications across components, API routes, configuration files, and database schema.

---
> Source: [fixit-labs/Lading-Page-](https://github.com/fixit-labs/Lading-Page-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
