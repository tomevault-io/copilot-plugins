## hono-simple-google-auth

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `hono-simple-google-auth` - a TypeScript library that provides Google OAuth authentication middleware for Hono web framework applications. The library supports JSX/TSX components and is designed to work across multiple platforms including Node.js, Cloudflare Workers, Bun, and Deno.

## Commands

**Build the project:**
```bash
npm run build
```

**Run tests:**
```bash
npm test
```

**Test and build (used before publishing):**
```bash
npm run prepublishOnly
```

## Architecture

### Core Components

- **`src/index.tsx`**: Main export with `honoSimpleGoogleAuth()` function that returns `{ routes, session }` object
  - `routes`: Hono sub-app with `/signin`, `/callback`, `/signout` endpoints
  - `session`: Middleware for extracting session data from cookies
- **`src/types.ts`**: TypeScript type definitions including `GoogleAuthEnv`, session types, and configuration interfaces
- **`src/GoogleSignInButton.tsx`**: Default TSX component for rendering Google Sign-In UI
- **`src/kvSessionStore.ts`**: Cloudflare Workers KV storage adapter for session management

### Authentication Flow

1. User visits `/auth/signin` → renders Google Sign-In button
2. Google posts credential to `/auth/callback` → verifies token and creates session
3. Session stored via configurable `sessionStore` interface
4. Session middleware extracts user data from cookies and makes it available as `c.var.session`

### Key Design Patterns

- **Provider function pattern**: Configuration is provided via async function `(c) => options` to support environment variables available only at request time
- **Platform agnostic**: Session storage is abstracted via interface, allowing different backends (KV, database, etc.)
- **Type-safe**: Uses `GoogleAuthEnv` type to extend Hono's context with session variables
- **Customizable UI**: Supports custom sign-in page components via `renderSignInPage` option

### Testing

Tests use Jest with ts-jest transformer. Test files are in `/test/` directory and cover authentication flows, session management, and basic auth scenarios.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kurrik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kurrik)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
