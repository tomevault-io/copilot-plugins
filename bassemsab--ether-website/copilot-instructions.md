## ether-website

> Generates a Svelte Playground link with the provided code.

# Ether Website Development Guide

## Project Setup

- Runtime: Bun
- Framework: SvelteKit with Svelte 5 (Runes)
- Database: Bun SQLite (`bun:sqlite`)
- Email: Resend
- Styling: Tailwind CSS
- Deployment: Bun Adapter

## Development Commands

```bash
# Install dependencies
bun install

# Start development server
bun run dev

# Type checking
bun run check

# Format code
bun run format

# Production build
bun run build

# Start production server
bun ./build/index.js
```

## Project Structure

- `src/routes/` - SvelteKit routes with [locale] internationalization
- `src/lib/components/` - Reusable Svelte 5 components
- `src/lib/server/` - Server-side code (database, email)
- `src/lib/stores/` - Svelte 5 state stores
- `src/lib/i18n/` - Internationalization configuration
- `build/` - Production output (generated)

## Key Features

- **Svelte 5 Runes**: Modern reactive syntax (`$state`, `$derived`, `$props`)
- **Internationalization**: Multi-language support with route-based locales
- **Admin Panel**: SQL interface for database management at `/admin`
- **Contact Forms**: Email integration via Resend
- **MCP Server**: Model Context Protocol endpoint at `/api/mcp`
- **Visitor Tracking**: SQLite database with visitor analytics

## Environment Variables

Configure these in `.env`:

```
RESEND_API_KEY=your_resend_api_key_here
RESEND_CONTACT_EMAIL=your_email@example.com
ADMIN_PASSWORD=ether-admin
DB_PATH=visitors.sqlite
```

## Svelte 5 Specific Notes

- All components use modern Runes syntax
- State management with `$state()` and `$derived()`
- Component props with `$props()`
- Effects with `$effect()`
- No legacy Svelte 4 syntax (no `export let`, no `$:`)

## Database

Uses Bun's native SQLite driver:

- File: `visitors.sqlite`
- Tables: `visitors` (IP, user-agent, path, timestamp)
- Admin interface at `/admin` for SQL queries
- **IMPORTANT**: Requires Bun runtime - no Node.js fallback

## Deployment

- Built for Bun runtime with `svelte-adapter-bun`
- Single executable: `./build/index.js`
- Environment-based configuration
- Ready for containerization (see `Dockerfile`)

## Code Quality

- TypeScript for type safety
- Prettier for code formatting
- ESLint configuration (needs v9 format)
- Svelte component auto-fixing via svelte-autofixer

## Agents Configuration

When working with AI agents, ensure they:

1. Use Svelte 5 Runes syntax exclusively
2. Understand Bun runtime specifics
3. Follow the established project structure
4. Use proper TypeScript types
5. Leverage existing components and utilities

## MCP Integration

The project includes a built-in MCP server for external tool integration:

- Endpoint: `/api/mcp`
- Authentication: Bearer token (ADMIN_PASSWORD)
- Features: Database queries, visitor logs
- Protocol: Server-Sent Events for real-time communication

---

## Svelte MCP Server Tools

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

---
> Source: [bassemsab/ether-website](https://github.com/bassemsab/ether-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
