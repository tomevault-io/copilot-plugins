## kel

> This file provides guidance when working with code in this repository.

This file provides guidance when working with code in this repository.

## Project Overview

Kel is an AI agent desktop application that lives on your computer. It's built as an Electron app with React (TypeScript) frontend and uses a local SQLite database via Drizzle ORM. The app appears as a sidebar window (similar to Raycast) that can be toggled with `Control+K`.

## Development Commands

```bash
# Install dependencies (uses pnpm)
pnpm install

# Development mode with hot reload
pnpm dev

# Type checking
pnpm typecheck              # Check both node and web
pnpm typecheck:node         # Check main/preload process only
pnpm typecheck:web          # Check renderer process only

# Code quality
pnpm lint
pnpm format

# Build for production
pnpm build                  # Build without packaging
pnpm build:mac             # Build and package for macOS
pnpm build:win             # Build and package for Windows
pnpm build:linux           # Build and package for Linux

# Database operations
pnpm db:generate           # Generate migrations from schema
pnpm db:migrate            # Run migrations
pnpm db:push               # Push schema changes directly
pnpm db:studio             # Open Drizzle Studio UI
```

## Architecture

### Electron Multi-Process Structure

The app follows standard Electron architecture with three processes:

1. **Main Process** (`src/main/`): Node.js backend
   - Entry point: `src/main/index.ts`
   - Manages the native window, global shortcuts (Control+K, Control+Option+K)
   - Runs database migrations on startup
   - Registers IPC handlers for renderer communication
   - Creates system tray icon (macOS)

2. **Preload Script** (`src/preload/`): Security bridge
   - `src/main/preload.ts` defines the actual preload logic
   - Exposes safe APIs to renderer via `contextBridge`
   - Type definitions in `src/preload/index.d.ts`

3. **Renderer Process** (`src/renderer/`): React frontend
   - Entry point: `src/renderer/src/main.tsx`
   - Uses React Router with hash routing
   - Routes defined in `src/renderer/src/routes/index.tsx`

### Path Aliases

Configured in `electron.vite.config.ts`:
- `@renderer`: Points to `src/renderer/src`
- `@shared`: Points to `src/preload/shared`

### Database Layer

- **ORM**: Drizzle ORM with SQLite (`@libsql/client`)
- **Location**: Database file stored in `.kel/database.db`
- **Configuration**: `drizzle.config.ts`
- **Schema**: `src/main/db/schema.ts` (re-exports from `tables/` subdirectory)

#### Database Tables

- `settings`: User preferences (name, API key, API provider type)
- `folders`: Tracked project folders with favorites
- `chat`: Conversation threads
- `message`: Individual messages within chats (references chat via foreign key)

#### Database Initialization

The app automatically runs migrations and initializes default settings on startup via `runMigrations()` and `initializeSettings()` in `src/main/index.ts`.

### IPC Communication Pattern

The app uses a handler-based IPC architecture:

1. **API Handlers** (`src/main/api/`): Each domain has its own subdirectory
   - `handlers.ts`: Implements IPC handlers using `ipcMain.handle()`
   - `schema.ts`: Zod schemas for validation and TypeScript types
   - Example: `src/main/api/chat/handlers.ts`

2. **Preload Exposure** (`src/main/preload.ts`): Wraps IPC calls in clean API
   - Example: `window.api.chat.list()` → `ipcRenderer.invoke('chat:list')`

3. **Renderer Hooks** (`src/renderer/src/hooks/`): React hooks wrap API calls
   - Example: `use-chat.ts` provides `useChats()` hook using React Query
   - Hooks handle loading states, caching, and mutations

4. **Registration**: All handlers registered via `registerAllApis()` in `src/main/api/index.ts`

### UI Components

- **Styling**: Tailwind CSS v4 with custom theme
- **Component Library**: Radix UI primitives + custom components in `src/renderer/src/components/ui/`
- **Forms**: React Hook Form with Zod validation (`@hookform/resolvers`)
- **Custom Titlebar**: `src/renderer/src/components/titlebar.tsx` with `use-titlebar.ts` hook

### Window Behavior

- **Position**: Sidebar-style window that docks to left or right edge of screen
- **Toggle**: `Control+K` shows/hides the window
- **Cycle Position**: `Control+Option+K` switches between left and right
- **Dimensions**: Fixed width (375-450px), full screen height
- **Appearance**: Transparent background with vibrancy effect (macOS)
- **Always on Top**: Stays above other windows, visible on all workspaces

### AI Integration

- Uses Vercel AI SDK (`ai` package)
- Supports multiple providers via `@openrouter/ai-sdk-provider`
- API keys stored in settings table with provider type (`openrouter` or `anthropic`)

## Important Notes

- **Package Manager**: Must use `pnpm` (see `pnpm` config in package.json)
- **Environment**: Copy `.env.example` to `.env` for local development
- **Database Changes**: After modifying schema, run `pnpm db:generate` then `pnpm db:push`
- **TypeScript**: Separate tsconfigs for node (`tsconfig.node.json`) and web (`tsconfig.web.json`)
- **Sandbox**: Preload runs with `sandbox: false` for filesystem access

## Code Patterns

When adding new features:

1. **New Database Table**:
   - Create table definition in `src/main/db/tables/`
   - Export from `src/main/db/schema.ts`
   - Create schemas in `src/main/api/{domain}/schema.ts`
   - Generate migration with `pnpm db:generate`

2. **New IPC API**:
   - Create handlers in `src/main/api/{domain}/handlers.ts`
   - Register in `src/main/api/index.ts`
   - Expose in `src/main/preload.ts`
   - Add types to `src/preload/index.d.ts`
   - Create React hook in `src/renderer/src/hooks/`

3. **New Route**:
   - Add page component to `src/renderer/src/routes/pages/`
   - Register route in `src/renderer/src/routes/index.tsx`

---
> Source: [not-manu/kel](https://github.com/not-manu/kel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
