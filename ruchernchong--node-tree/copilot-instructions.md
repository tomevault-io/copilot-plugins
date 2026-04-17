## node-tree

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

- `bun dev` - Start development server
- `bun run build` - Build for production
- `bun run lint` - Run Biome linter
- `bun run format` - Format code with Biome
- `bun run db:push` - Push database schema changes
- `bun run db:studio` - Open Drizzle Studio for database management
- `bun run db:generate` - Generate database migrations
- `bun run db:migrate` - Run database migrations
- `bun run auth:generate` - Generate Better Auth types

## Code Style Guidelines

### Tailwind CSS v4 Styling
- **Never use `margin-top`** (`mt-*` classes)
- **Use `gap-*` instead of `space-y-*`** for vertical spacing
- **Only use even numbers** for gap values (gap-2, gap-4, gap-6, etc.)

### TypeScript Conventions
- **Use arrow functions** for function declarations
- **Prefer `interface` over `type`** when possible

### Naming Conventions
- **Use kebab-case** for all filenames

## Architecture Overview

### Multi-Tenant Architecture

The project uses a **single central Turso database** with user-scoped queries. All data is stored in one database (`TURSO_DATABASE_URL`), and isolation happens at the query level using `userId` foreign key references.

**Important Notes:**
- Directory structure includes `src/schema/user/` and `src/schema/central/` for future per-user database expansion, but currently only the central database is used
- Actions currently use `HARDCODED_USER_ID = "temp-user-id"` in `src/actions/links.ts` - this needs replacement with actual session-based user ID once authentication is fully integrated
- All database operations must scope queries by `userId` to maintain data isolation

### Database Schema Organization

All schemas are defined in `src/db/schema.ts`:

**Authentication Tables** (managed by Better Auth):
- `user` - User profiles with OAuth metadata
- `session` - Authentication sessions
- `account` - OAuth provider accounts (GitHub, Google)
- `verification` - Email verification tokens
- `passkey` - WebAuthn credentials

**Application Tables:**
- `link` - Custom links with scheduling support
  - Fields: slug, title, url, icon, description, category, isActive, startDate, endDate, order
  - Supports temporal availability via `startDate`/`endDate`
  - Drag-and-drop reordering via `order` field
- `profileSettings` - User display preferences (displayName, bio, theme, customStyles)
- `clickEvent` - Privacy-focused analytics
  - Uses `userHash` instead of raw IP addresses (hashed with `IP_SALT` environment variable)
  - Tracks: referrer, device, browser, country, clickedAt timestamp
  - No PII stored

**Schema Patterns:**
- All timestamps use `timestamp_ms` mode for precision
- Automatic `createdAt`/`updatedAt` management via Drizzle ORM
- Cascade deletes on user removal
- Named exports from schema prevent circular imports

### Authentication System

**Server-Side** (`src/lib/auth.ts`):
- Better Auth with Drizzle adapter for SQLite
- OAuth providers: GitHub (required) and Google (optional)
- Plugins: `oAuthProxy()`, `passkey()`, `lastLoginMethod()`
- Email/password authentication enabled

**Client-Side** (`src/lib/auth-client.ts`):
- Uses `createAuthClient` for React hooks
- Plugins: passkey and lastLoginMethod client variants
- Base URL configured via `BETTER_AUTH_URL`

**Authentication Flow:**
- API routes at `/api/auth/[...all]` delegate to Better Auth handlers
- Middleware protection in `src/proxy.ts` checks authentication
- Admin routes protected via `ADMIN_USER_ID` environment variable
- Type-safe session/user types in `src/lib/auth/types.ts`

### Colocation Pattern (Next.js App Router)

Route-specific code lives adjacent to routes in **private folders** (prefix with `_`):

```
src/app/
├── (admin)/              # Admin route group
│   └── dashboard/
│       ├── _components/  # Admin-only components
│       ├── _lib/        # Admin-only data fetching
│       ├── _utils/      # Admin-only utilities
│       └── _hooks/      # Admin-only hooks
├── (public)/            # Public route group
│   └── _components/     # Public-only components
└── api/                 # API routes
```

**Global Shared Code:**
- `src/components/ui/` - shadcn/ui components (Button, Input, Label, Switch, etc.)
- `src/components/links/` - Shared link components (LinkForm, SortableLinksList)
- `src/components/layout/` - Global layout components
- `src/lib/` - Shared utilities and configurations

**Route Groups:**
- `(admin)` - Protected dashboard routes at `/dashboard/*`
  - `/dashboard` - Overview with stats
  - `/dashboard/links` - Link management with drag-and-drop
  - `/dashboard/links/new` - Create link form
  - `/dashboard/links/[id]/edit` - Edit link form
- `(public)` - Public-facing routes (landing page, user profiles)

### Data Flow & Validation

**Mutation Pattern:**
1. Client Component (form with react-hook-form) validates using Zod schema
2. Calls Server Action (in `src/actions/`)
3. Server Action performs database operation via Drizzle ORM
4. Server Action calls `revalidatePath()` for automatic cache invalidation
5. Client receives response and shows toast notification (Sonner)

**Validation Layers:**
- Zod schemas in `src/lib/schemas/` for runtime validation
- TypeScript types inferred from schemas (`z.infer<typeof schema>`)
- Server Actions validate input before database operations
- Form validation with react-hook-form and `@hookform/resolvers/zod`

**Server Actions** (`src/actions/links.ts`):
- Use `"use server"` directive
- Functions: `createLink`, `updateLink`, `deleteLink`, `toggleLinkActive`, `reorderLinks`
- Auto-revalidate cache with `revalidatePath()` after mutations
- All operations scoped to `HARDCODED_USER_ID` (needs session integration)

### Key Development Patterns

**Component Architecture:**
- **Server Components** (default) - For layouts, pages, and data fetching
- **Client Components** (`"use client"`) - For forms, interactive features, and state management
- Async Server Components fetch data at render time
- Optimistic UI updates in interactive components (e.g., SortableItem)

**Database Operations:**
- Drizzle ORM for type-safe queries
- All operations must be scoped to user via `eq(link.userId, userId)`
- Composable query builders with filters (`and`, `eq`, `desc`, `asc`)
- Use `db.select()`, `db.insert()`, `db.update()`, `db.delete()` for CRUD operations

**Error Handling:**
- Try-catch blocks in Server Actions
- User-friendly error messages via Sonner toasts
- Type-safe error responses from Server Actions

**Type Safety:**
- Better Auth types inferred from auth instance
- Drizzle types auto-generated from schema
- Zod runtime validation ensures type safety at boundaries

### Link Scheduling & Analytics

**Scheduling:**
- Links have optional `startDate` and `endDate` fields
- Validation ensures `endDate > startDate` via Zod schema
- Admin can toggle `isActive` status for immediate enable/disable
- Scheduled links automatically become available/unavailable based on date range

**Analytics:**
- Privacy-focused: Uses `userHash` instead of IP addresses
- IP hashing via `IP_SALT` environment variable
- Tracks: referrer, device, browser, country, clickedAt timestamp
- No PII stored - GDPR compliant
- All analytics stored in `clickEvent` table for easy reporting

## Environment Variables

See `.env.example` for full list. Key variables:

**Required:**
- `BETTER_AUTH_SECRET` - Auth encryption key (generate with `openssl rand -base64 32`)
- `TURSO_DATABASE_URL` - Turso database URL
- `TURSO_AUTH_TOKEN` - Turso auth token
- `UPSTASH_REDIS_REST_URL` - Redis URL for caching
- `UPSTASH_REDIS_REST_TOKEN` - Redis token
- `GITHUB_CLIENT_ID` / `GITHUB_CLIENT_SECRET` - GitHub OAuth credentials
- `IP_SALT` - Salt for IP hashing (generate with `openssl rand -base64 32`)

**Optional:**
- `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` - Google OAuth credentials
- `ADMIN_USER_ID` - User ID with admin access
- `ADMIN_USERNAME` - Admin username for display

## Deployment

- Vercel deployment with edge runtime
- Build command: `bun run vercel-build` (runs migrations then build)
- Turso provides edge-accessible SQLite
- Update OAuth callback URLs for production domain
- Run database migrations before deploying

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ruchernchong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
