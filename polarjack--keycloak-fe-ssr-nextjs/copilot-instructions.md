## keycloak-fe-ssr-nextjs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 application bootstrapped with `create-next-app`, using the App Router architecture. The project is an MPC (Multi-Party Computation) MVP frontend with **Server-Side Rendering (SSR)** and custom Keycloak authentication.

**Tech Stack:**
- Next.js 15.5.9 with Turbopack
- React 19.1.0
- TypeScript 5
- Tailwind CSS 4 (using the new `@tailwindcss/postcss` plugin)
- pnpm workspace
- jose 6.1.3 for JWT verification
- Custom SSR authentication with Keycloak (OAuth2/OIDC)

## Commands

### Development
```bash
pnpm dev          # Start development server with Turbopack (http://localhost:3000)
pnpm build        # Build for production with Turbopack
pnpm start        # Start production server
pnpm lint         # Run ESLint
```

Note: This project uses **pnpm** as the package manager, not npm or yarn.

### Turbopack
All build and dev commands use `--turbopack` flag by default. Turbopack is Next.js's Rust-based bundler for faster builds and hot reloading.

## Architecture

### Project Structure
```
src/
  app/               # Next.js App Router directory
    page.tsx         # Home page (SSR, route: /)
    layout.tsx       # Root layout with fonts and metadata
    globals.css      # Global styles with Tailwind imports
    profile/         # User profile page (protected, SSR)
      page.tsx       # Profile page (Server Component)
      components/    # SSR components for profile page
        TokenDisplay.tsx
        ProfileActions.tsx
    api/
      auth/          # Authentication API routes
        login/route.ts      # Initiate OAuth2 flow
        callback/route.ts   # Handle OAuth2 callback
        logout/route.ts     # Logout endpoint
        refresh/route.ts    # Token refresh endpoint
    favicon.ico
  lib/               # Utility libraries
    auth.ts          # JWT verification, Keycloak endpoints
    cookies.ts       # Cookie management utilities
  middleware.ts      # Route protection and token validation
public/              # Static assets
```

### Path Aliases
- `@/*` maps to `./src/*` - use this for all internal imports

### Styling
- **Tailwind CSS 4** with the new PostCSS plugin (`@tailwindcss/postcss`)
- Uses `@theme inline` directive in globals.css for CSS variable configuration
- Custom CSS variables for theming: `--background`, `--foreground`, `--font-geist-sans`, `--font-geist-mono`
- Automatic dark mode support via `prefers-color-scheme`

### Fonts
- Uses `next/font/google` to load Geist and Geist Mono fonts
- Fonts are configured in layout.tsx with CSS variables:
  - `--font-geist-sans`
  - `--font-geist-mono`

### TypeScript Configuration
- Strict mode enabled
- Module resolution: `bundler`
- Target: ES2017
- Path alias: `@/*` → `./src/*`

### ESLint
- Uses flat config format (eslint.config.mjs)
- Extends `next/core-web-vitals` and `next/typescript`
- Compatible with Next.js TypeScript best practices

## Key Implementation Details

### App Router
This project uses Next.js App Router (not Pages Router). Components in `src/app/` are Server Components by default unless marked with `'use client'`.

### Tailwind CSS 4
This project uses Tailwind CSS v4 which has a different configuration approach:
- No `tailwind.config.ts` file (removed in v4)
- Configuration is done via `@theme` directive in CSS files
- PostCSS plugin: `@tailwindcss/postcss`

### CSS Variables Pattern
The project uses a CSS variable system for theming:
- Root variables defined in `:root` and `@media (prefers-color-scheme: dark)`
- Variables registered in `@theme inline` block
- Referenced in Tailwind classes (e.g., `bg-background`, `text-foreground`)

## Authentication

### SSR Keycloak Integration
This project uses a **custom Server-Side Rendering (SSR) authentication** implementation with Keycloak. Unlike Auth.js (NextAuth), this implementation maintains Keycloak's JWT tokens as the single source of truth.

**Why Custom SSR Instead of Auth.js?**
- Keycloak JWT tokens are used directly (no dual-session management)
- No synchronization issues between web app and Keycloak
- Direct integration with backend services using Keycloak tokens
- Full control over token lifecycle

**Setup:**
1. Copy `.env.example` to `.env.local`
2. Configure environment variables:
   - `NEXT_PUBLIC_KEYCLOAK_URL` - Keycloak server URL
   - `NEXT_PUBLIC_KEYCLOAK_REALM` - Keycloak realm name
   - `NEXT_PUBLIC_KEYCLOAK_CLIENT_ID` - Client ID for this application
   - `KEYCLOAK_CLIENT_SECRET` - Client secret (confidential - server-side only)
   - `NEXTAUTH_URL` - Base URL of the Next.js app (for OAuth callbacks)

3. Configure Keycloak client:
   - **Access Type**: `confidential` (for SSR backend authentication)
   - **Valid Redirect URIs**: `http://localhost:3000/api/auth/callback`
   - **Web Origins**: `http://localhost:3000`
   - Copy the generated **Client Secret** from Keycloak to your `.env.local` file

**Architecture:**

1. **OAuth2/OIDC Flow:**
   - Server-side authorization code flow with **confidential client** (client secret)
   - Tokens stored in HTTP-only cookies (XSS protection)
   - Next.js middleware validates tokens on every request

2. **API Routes:**
   - `/api/auth/login` - Initiates OAuth2 authorization
   - `/api/auth/callback` - Exchanges code for tokens using client secret, sets cookies
   - `/api/auth/logout` - Clears cookies, redirects to Keycloak logout
   - `/api/auth/refresh` - Refreshes access token using refresh token and client secret

3. **Middleware (`src/middleware.ts`):**
   - Protects routes (e.g., `/profile`)
   - Verifies JWT signature and expiration using JWKS
   - Automatically refreshes expired tokens
   - Redirects to login if authentication fails

4. **Token Storage:**
   - HTTP-only cookies (not accessible from JavaScript)
   - Cookies: `access_token`, `refresh_token`, `id_token`
   - Secure flag in production (HTTPS only)

**Features:**
- SSO (Single Sign-On) with Keycloak
- Server-Side Rendering with authentication
- Automatic token refresh (on-demand via middleware)
- JWT token verification using Keycloak's JWKS
- Protected routes via middleware
- Direct token access in Server Components

**Routes:**
- `/` - Home page (SSR)
  - Shows "Sign In" button for unauthenticated users
  - Automatically redirects authenticated users to `/profile`
- `/profile` - Protected profile page (SSR, requires authentication)
  - Displays JWT tokens (access, refresh, ID)
  - Shows decoded token payloads with syntax highlighting
  - Shows token expiration status (expired or valid with timestamp)
  - Automatic token refresh via middleware (no manual refresh needed)
  - Logout button (redirects to `/api/auth/logout`)

**Backend Integration:**
You can use Keycloak tokens directly in Server Components to call backend APIs:

```typescript
// Server Component
import { getAuthTokens } from '@/lib/cookies';

export default async function DataPage() {
  const { accessToken } = await getAuthTokens();

  const response = await fetch('https://backend-api.com/data', {
    headers: { 'Authorization': `Bearer ${accessToken}` }
  });

  const data = await response.json();
  return <div>{/* render data */}</div>;
}
```

**Key Implementation Files:**
- `src/lib/auth.ts` - JWT verification, JWKS fetching, Keycloak endpoints
- `src/lib/cookies.ts` - Cookie management (set, get, delete tokens)
- `src/middleware.ts` - Route protection and token validation
- `src/app/api/auth/*` - OAuth2 flow handlers

**Important Implementation Notes:**

1. **JWT Verification (Keycloak-specific):**
   - The `verifyToken()` function in `src/lib/auth.ts` does NOT validate the `aud` (audience) claim
   - This is intentional for Keycloak compatibility, as Keycloak access tokens typically have `aud: "account"` instead of the client ID
   - Security is maintained through:
     - JWT signature verification using Keycloak's JWKS (public keys)
     - Issuer verification (ensures token is from correct Keycloak realm)
     - Expiration check (automatically handled by `jose` library)
   - **Do not add audience validation** for access tokens - it will break authentication

2. **Logout Flow:**
   - Keycloak's logout endpoint requires the `id_token_hint` parameter
   - The logout route (`src/app/api/auth/logout/route.ts`) retrieves the ID token from cookies before clearing them
   - If no ID token is available, the logout skips the Keycloak logout redirect and only clears local cookies
   - The `openid` scope in the login request ensures an ID token is returned

3. **Token Scopes:**
   - Login request must include `scope: 'openid profile email'`
   - The `openid` scope is required to receive an ID token (needed for logout)
   - Without `openid`, only access and refresh tokens are returned

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/polarjack)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/polarjack)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
