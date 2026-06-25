## neurosync

> Generates a Svelte Playground link with the provided code.

# Project: VANTA

SvelteKit 5 app (runes, TypeScript strict, Node adapter) with Supabase auth/DB,
Upstash Redis caching, and OpenRouter LLM integration. MVP sprint.

## Commands

- `npm run dev` — Dev server
- `npm run build` — Production build
- `npm run check -- --output machine` — TypeScript check
- `npm test` — Playwright E2E tests
- `npm run format` — Format with Prettier

## Architecture

- `/src/routes` — SvelteKit file-based routing (+page.svelte, +page.server.ts)
- `/src/lib/components` — Reusable Svelte 5 components
- `/src/lib/server` — Server-only: Supabase client, Redis, OpenRouter client
- `/src/lib/utils` — Shared utilities and types
- `/supabase/migrations` — Database migrations

## IMPORTANT

- Svelte 5 ONLY: use $state(), $derived(), $effect(), $props() — NEVER legacy reactive syntax
- Event handlers: `onclick` not `on:click`; `oninput` not `on:input`
- Supabase: always use server-side load functions for authenticated queries
- Redis ops in $lib/server/redis.ts ONLY — never client-side
- OpenRouter calls in $lib/server/openrouter.ts — stream responses via server-sent events
- NEVER commit .env; NEVER disable Row Level Security
- Run `npm run check` before every commit

## Code Style

- Named exports only, no default exports
- TypeScript strict — no `any` types
- Tailwind utility classes, no custom CSS files
- Components under 200 lines; extract when larger
- camelCase variables, PascalCase components

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

## Available MCP Tools:

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
> Source: [neurosyncapp/neurosync](https://github.com/neurosyncapp/neurosync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
