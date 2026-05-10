## hitparade

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hitparade — a SvelteKit web application using Svelte 5 (runes mode), Tailwind CSS 4, and Flowbite Svelte as the UI component library.

## Docker

The project runs in Docker. **Always run commands inside the container, not on the host.**

```bash
# Development (hot reload at localhost:5173):
docker compose --profile dev up --build

# Production (localhost:3000):
docker compose --profile prod up --build

# Run a command inside the dev container:
docker compose --profile dev exec hitparade-dev <command>
```

## Commands (run inside the container)

- **Dev server**: `bun run dev` (started automatically by the dev container)
- **Build**: `bun run build`
- **Preview production build**: `bun run preview`
- **Type check**: `bun run check`
- **Lint**: `bun run lint` (runs prettier --check + eslint)
- **Format**: `bun run format`

## Tech Stack

- **Framework**: SvelteKit with `adapter-node`
- **Language**: TypeScript (strict runes mode enforced in `svelte.config.js`)
- **Styling**: Tailwind CSS 4 via Vite plugin (not PostCSS), with `@tailwindcss/forms`
- **UI Components**: Flowbite Svelte + Flowbite Svelte Icons
- **Package Manager**: bun
- **Formatting**: Prettier with tabs, single quotes, no trailing commas, 100 char width

## Architecture

Standard SvelteKit file-based routing under `src/routes/`. Shared code goes in `src/lib/`. Global styles in `src/routes/layout.css` (Tailwind entry point with Flowbite plugin and dark mode variant).

## Svelte 5 Conventions

- All components use runes mode (`$props()`, `$state()`, `$derived()`, `$effect()`, etc.)
- Use `{@render children()}` for slot content, not `<slot>`
- The runes config in `svelte.config.js` only applies runes mode to project files, not `node_modules`

## MCP Tools

### Svelte/SvelteKit

The Svelte MCP server is available. When working on Svelte/SvelteKit code:

1. Use `list-sections` first to discover relevant documentation
2. Use `get-documentation` to fetch the sections you need
3. Use `svelte-autofixer` to validate any Svelte code before finalizing

### Context7

For Flowbite Svelte components, use the Context7 MCP server to get usage examples and API details.

---
> Source: [jeroentvb/hitparade](https://github.com/jeroentvb/hitparade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
