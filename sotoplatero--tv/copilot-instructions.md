## tv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a SvelteKit application designed to display dashboard-style pages on TV screens. The primary purpose is to show real-time information on large monitors/TVs in a manufacturing environment.

The application connects to an Odoo ERP system via JSON-RPC to display manufacturing work order tracking data. It's built to run continuously on TV displays with automatic refresh and optimized for viewing from a distance.

## Package Manager

This project uses **pnpm** (not npm or yarn). Always use pnpm for package management.

## Development Commands

```bash
# Start development server
pnpm dev

# Start dev server and open in browser
pnpm dev -- --open

# Build for production
pnpm build

# Preview production build
pnpm preview

# Type checking
pnpm check

# Type checking in watch mode
pnpm check:watch
```

## Architecture

### Tech Stack
- **Framework**: SvelteKit 2 with Svelte 5 (using runes: `$state`, `$props`, etc.)
- **Styling**: Tailwind CSS 4 (via Vite plugin)
- **Build Tool**: Vite
- **Type Checking**: TypeScript/JSDoc (jsconfig.json with checkJs enabled)
- **Deployment**: Uses @sveltejs/adapter-node for Dokku deployment

### Key Application Structure

**Core Functionality:**
- Main route (`/routes/+page.svelte`): Basic welcome page
- Odoo route (`/routes/odoo/`): TV dashboard displaying active manufacturing orders
  - `+page.server.ts`: Server-side data fetching from Odoo
  - `+page.svelte`: Full-screen TV-optimized UI with auto-refresh

**Odoo Integration (`src/lib/odoo-client.js`):**
- Custom OdooClient class for JSON-RPC communication
- Authenticates with Odoo and fetches data from `mrp.workcenter.productivity` model
- Methods: `authenticate()`, `searchRead()`, `read()`, `search()`
- Uses `createOdooClient()` factory function that imports environment variables directly from `$env/static/private`
- Validates required environment variables on client creation

**Environment Variables:**
Required variables in `.env`:
- `ODOO_URL`: Odoo instance URL
- `ODOO_DATABASE`: Odoo database name
- `ODOO_USERNAME`: Odoo API username
- `ODOO_PASSWORD`: Odoo API password

Access via SvelteKit's `$env/static/private` (server-side only).

### TV Display Features
- Auto-refresh every 30 seconds (full page reload)
- Wake Lock API to prevent screen sleep
- Large text sizes optimized for TV viewing (text-6xl to text-8xl)
- Airport-style table layout with status indicators
- Shows: work order name, employee, work center, start time, time remaining, status badges
- Color-coded status: Productive (green), Performance (yellow), Availability (red)

## Deployment

### Dokku Deployment
The application is configured for deployment to Dokku using Docker:

1. **Dockerfile**: Multi-stage build using Node.js 20 Alpine
   - Build stage: Installs dependencies and builds the application
   - Production stage: Only production dependencies and built application
   - Exposes port 3000

2. **Environment Variables**: Set these in Dokku before deploying:
   ```bash
   dokku config:set APP_NAME ODOO_URL=https://your-odoo.com
   dokku config:set APP_NAME ODOO_DATABASE=your_db
   dokku config:set APP_NAME ODOO_USERNAME=your_user
   dokku config:set APP_NAME ODOO_PASSWORD=your_password
   ```

3. **Deploy**: Push to Dokku git remote:
   ```bash
   git push dokku main
   ```

## Important Notes

**MCP Server:** Svelte MCP server is configured in `.mcp.json` for enhanced Svelte development assistance.

**Path Aliases:**
- `$lib` → `src/lib`
- Standard SvelteKit path aliases apply

## Coding Patterns

- Use Svelte 5 runes syntax (`$state`, `$props`, `$derived`, etc.)
- Server-side data loading via `+page.server.ts` with `PageServerLoad` type
- TypeScript types imported from `./$types` (SvelteKit generated)
- Component props use `let { data }: { data: PageData } = $props()`

---
> Source: [sotoplatero/tv](https://github.com/sotoplatero/tv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
