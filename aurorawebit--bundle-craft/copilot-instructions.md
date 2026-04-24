## bundle-craft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**bundle-craft** is an embedded Shopify app built with React Router v7 and the `@shopify/shopify-app-react-router` package. It uses Prisma (SQLite) for session storage and the Shopify Admin GraphQL API. The UI uses **Polaris web components** (`<s-page>`, `<s-button>`, `<s-section>`, etc.), not React Polaris components.

## Commands

- `npm run dev` — Start local dev server via Shopify CLI (runs `shopify app dev`, which handles tunneling, env vars, and Prisma migrations)
- `npm run build` — Production build (`react-router build`)
- `npm run lint` — ESLint with cache
- `npm run typecheck` — Generate React Router types then run `tsc --noEmit`
- `npm run setup` — Generate Prisma client and run migrations (`prisma generate && prisma migrate deploy`)
- `npm run graphql-codegen` — Generate TypeScript types from GraphQL operations
- `npm run deploy` — Deploy app config and extensions to Shopify (`shopify app deploy`)
- `npx prisma migrate dev` — Create a new migration after schema changes
- `npx prisma studio` — Open Prisma database browser

## Architecture

### Routing

File-system routing via `@react-router/fs-routes` (flat routes convention). Routes live in `app/routes/`:

- `app.tsx` — Layout for all authenticated `/app/*` pages. Wraps children in `<AppProvider>` and renders the app nav. Every nested `app.*` route gets Shopify admin authentication via `authenticate.admin(request)` in its loader.
- `app._index.tsx` — Main app page (demo product creation via Admin GraphQL)
- `app.additional.tsx` — Example secondary page
- `auth.$.tsx` — Catch-all auth route (handles OAuth callbacks)
- `auth.login/route.tsx` — Login page (non-embedded)
- `_index/route.tsx` — Public landing page (redirects to `/app` if shop param present)
- `webhooks.app.*.tsx` — Webhook handlers (one file per topic)

### Key Files

- `app/shopify.server.ts` — Shopify app configuration singleton. Exports `authenticate`, `login`, `unauthenticated`, `sessionStorage`, etc. All server-side Shopify interactions go through this.
- `app/db.server.ts` — Prisma client singleton (uses global var in dev to survive HMR)
- `app/entry.server.tsx` — SSR entry point with streaming support
- `prisma/schema.prisma` — Database schema (SQLite, currently only `Session` model)
- `shopify.app.toml` — Shopify app config (scopes, webhooks, API version)

### Conventions

- **Authentication**: Call `await authenticate.admin(request)` in loaders/actions for app routes. Use `await authenticate.webhook(request)` in webhook routes.
- **GraphQL**: Use `admin.graphql()` from the authenticated context. Prefix queries with `#graphql` for syntax highlighting. Types are generated to `app/types/`.
- **Navigation**: Use `<s-link>` or React Router's `Link`—never raw `<a>` tags in embedded context. Use `redirect` from `authenticate.admin`, not from `react-router`.
- **Webhooks**: Defined declaratively in `shopify.app.toml`, each with a corresponding route file in `app/routes/webhooks.*.tsx`.
- **Extensions**: Live in `extensions/` directory (npm workspaces). Currently empty but scaffolded via `npm run generate`.

### Tech Stack

- React Router v7 (Vite plugin), React 18, TypeScript
- `@shopify/shopify-app-react-router` (server + react)
- `@shopify/app-bridge-react` for embedded app bridge
- Prisma 6 with SQLite
- Shopify Admin GraphQL API (version October25)
- Shopify CLI for dev/deploy
- ESLint 8, Prettier
- Node >=20.19

### MCP

The Shopify Dev MCP server is configured in `.mcp.json` for GraphQL schema introspection and docs access during development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aurorawebit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
