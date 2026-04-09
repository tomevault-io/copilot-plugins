## chronos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chronos is a modern calendar application with both web and desktop (Electron) versions. It integrates with Google Calendar and provides a minimalist UI for managing events. The project uses a monorepo structure managed by Turbo.

**Tech Stack:**

- **Web App**: React Router (formerly Remix), Supabase, Tailwind CSS, shadcn/ui
- **Desktop App**: Electron wrapper for the web app
- **Build Tool**: Turbo for monorepo management
- **Package Manager**: pnpm

## Development Commands

### Root Level (Monorepo)

```bash
# Run both web and desktop apps in development
pnpm dev:desktop

# Run only the web app
pnpm dev:web

# Run only the desktop app (Electron)
pnpm dev:electron

# Build all apps
pnpm build

# Build desktop app specifically
pnpm build:desktop
```

### Web App (apps/web)

```bash
# Development server (port 5174)
pnpm dev

# Run tests
pnpm test        # Run once
pnpm test:watch  # Watch mode

# Code quality
pnpm lint        # ESLint
pnpm typecheck   # TypeScript type checking
pnpm format      # Prettier formatting

# Production build
pnpm build
pnpm start       # Start production server
```

### Desktop App (apps/desktop)

```bash
# Development (requires web app running on port 5174)
pnpm electron:dev

# Build Electron app
pnpm electron:build

# Package for distribution
pnpm dist
```

## Architecture

### Monorepo Structure

- `/apps/web/` - React Router web application
- `/apps/desktop/` - Electron desktop wrapper
- `/packages/` - Shared packages (if any)
- `turbo.json` - Turbo build pipeline configuration
- `pnpm-workspace.yaml` - pnpm workspace configuration

### Web App Architecture

**Routing**: React Router with file-based routing

- Routes are in `/apps/web/app/routes/`
- API routes: `api.*.ts` files
- Page routes: `*.tsx` files
- Layout routes: `_calendar.tsx` (parent layout)

**State Management**:

- Server-side data loading via React Router loaders/actions
- Client-side state with React hooks
- Supabase for authentication and real-time data

**Styling**:

- Tailwind CSS for utility-first styling
- shadcn/ui components (installed via `pnpm dlx shadcn@latest add [component]`)
- Dark/light theme support

**Key Libraries**:

- `@supabase/supabase-js` - Authentication and database
- `@googleapis/calendar` - Google Calendar API integration
- `openai` - AI scheduling features
- `date-fns` - Date manipulation
- `lucide-react` - Icons

### Desktop App Architecture

- Simple Electron wrapper that loads the web app
- Configuration in `apps/desktop/src/main.ts`
- Supports protocol handling (`chronos://`)
- Multi-platform build configuration (Mac, Windows, Linux)

## Important Patterns

### API Routes

All API endpoints follow the pattern `api.{resource}.{action}.ts`:

- `api.calendar.events.ts` - Fetch calendar events
- `api.calendar.update-event.ts` - Update events
- `api.calendar.schedule-with-ai.ts` - AI scheduling

### Authentication Flow

1. User authenticates via Supabase (Google OAuth)
2. Session managed by Supabase SSR
3. Protected routes check authentication in loaders
4. Callback handled at `/auth/callback`

### Component Organization

- UI components: `/app/components/ui/` (shadcn/ui components)
- Feature components: `/app/components/` (business logic)
- Utilities: `/app/lib/` (helper functions, API clients)

## Key Considerations

### React Router Migration

The project recently migrated from Remix to React Router. Key differences:

- Use `react-router` commands instead of `remix`
- Routes configuration in `app/routes.ts`
- Entry points: `entry.client.tsx` and `entry.server.tsx`

### React Router v7 Response Utilities

**IMPORTANT**: React Router v7 uses `data()` instead of `json()` for responses:

```typescript
// ❌ WRONG - This will cause import errors
import { json } from "react-router";
return json({ message: "Hello" }, { status: 200 });

// ✅ CORRECT - Use data() in React Router v7
import { data } from "react-router";
return data({ message: "Hello" }, { status: 200 });
```

- `data()` - Returns JSON responses (replaces Remix's `json()`)
- `redirect()` - Still available for redirects
- Both are imported from `"react-router"` not `"@react-router/node"`

### Environment Variables

Required for web app (in `.env`):

- `VITE_SUPABASE_URL` - Supabase project URL
- `VITE_SUPABASE_ANON_KEY` - Supabase anonymous key
- Additional Google OAuth and OpenAI keys as needed

### Testing

- Test framework: Vitest
- Test files: `*.test.ts` or `*.test.tsx`
- Setup file: `vitest.setup.ts`
- Run tests before committing changes

### Code Style

- TypeScript for type safety
- Functional components with hooks
- Minimal state, prefer server-side data
- Simple, readable code over clever solutions
- Follow existing patterns in the codebase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kennethkoontz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kennethkoontz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
