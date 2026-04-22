## watermelon

> A link-saving and organizing application that lets users save, categorize, and search web links across web, browser extension, and mobile platforms. Built as a Turborepo monorepo with three frontends sharing a single Convex backend.

# Watermelon - Link Saving App

## Overview

A link-saving and organizing application that lets users save, categorize, and search web links across web, browser extension, and mobile platforms. Built as a Turborepo monorepo with three frontends sharing a single Convex backend.

## Tech Stack

- **Package manager**: Bun
- **Monorepo**: Turborepo
- **Web**: Next.js 15 (App Router) + TailwindCSS
- **Chrome Extension**: React + Vite (Manifest V3)
- **Mobile**: Expo (React Native) + NativeWind
- **Backend**: Convex (shared via `packages/convex`)
- **Auth**: Convex Auth (email/password + Google OAuth)
- **Testing**: Vitest + Testing Library
- **Language**: TypeScript (strict mode)

## Project Structure

```
watermelon/
├── apps/
│   ├── web/           # Next.js 15 web app
│   ├── extension/     # Chrome extension (React + Vite)
│   └── mobile/        # Expo React Native app
├── packages/
│   ├── convex/        # Shared Convex schema + functions
│   ├── eslint-config/ # Shared ESLint configurations
│   ├── typescript-config/ # Shared tsconfig bases
│   └── ui/            # Shared types and constants
```

## Commands

```bash
bun install              # Install all dependencies
bun run build            # Build all packages
bun run dev              # Dev mode for all apps
bun run lint             # Lint all packages
bun run test             # Run all tests
bun run typecheck        # Type-check all packages
bun run dev:web          # Dev mode for web only
bun run dev:extension    # Dev mode for extension only
bun run dev:mobile       # Dev mode for mobile only
bun run format           # Format with Prettier
bun run format:check     # Check formatting
```

## Key Patterns

### Consuming Convex from apps

```typescript
import { api } from "@watermelon/convex/api";
import { useQuery, useMutation } from "convex/react";
const links = useQuery(api.links.getUserLinks, { filter: "all" });
```

### Auth check in Convex functions

Every mutation/query that needs auth calls `getAuthenticatedUser(ctx)` which throws if not authenticated.

### Conditional queries

```typescript
const results = useQuery(
  api.links.searchLinks,
  searchTerm.length > 0 ? { searchTerm } : "skip",
);
```

## Environment Variables

- `packages/convex/.env.local`: `CONVEX_DEPLOYMENT`
- `apps/web/.env.local`: `NEXT_PUBLIC_CONVEX_URL`
- `apps/extension/.env`: `VITE_CONVEX_URL`
- `apps/mobile/.env`: `EXPO_PUBLIC_CONVEX_URL`
- Convex Dashboard: `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anuragts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
