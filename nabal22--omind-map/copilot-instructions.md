## omind-map

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A rap media mobile-first site with a 3D globe showing artist locations worldwide, articles, and playlists — built with SvelteKit 2, Svelte 5, TypeScript, Threlte (Three.js), and Tailwind CSS v4. Uses Bun as the package manager. Deployed on Cloudflare.

## Commands

- `bun run dev` — Start dev server
- `bun run build` — Production build
- `bun run preview` — Preview production build
- `bun run check` — Type-check with svelte-check
- `bun run lint` — Run Prettier and ESLint checks
- `bun run format` — Auto-format with Prettier

## Architecture

- **Routing**: SvelteKit file-based routing in `src/routes/`
- **Components/utilities**: `src/lib/` (aliased as `$lib`)
- **3D rendering**: Threlte (`@threlte/core`, `@threlte/extras`) wrapping Three.js
- **Globe**: Country borders rendered from Natural Earth 110m GeoJSON (`static/data/ne_110m_countries.geojson`)
- **Country geometries**: Use geojson and earcut
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` plugin — config uses `@theme` directive in `src/routes/layout.css` (no `tailwind.config` file)
- **Adapter**: `adapter-cloudflare` for Cloudflare Pages deployment
- **Static assets**: Icons, GeoJSON data, and web manifest in `static/`

## Code Conventions

- **Svelte 5 runes**: Use `$props()`, `$state()`, `$derived()`, `$effect()` — not legacy Svelte 4 syntax
- **Snippets over slots**: Use `{@render children()}` pattern, not `<slot />`
- **Tailwind utility-first**: Use Tailwind classes directly in markup — no `<style>` blocks with custom CSS classes
- **Tailwind theme**: Custom values defined via `@theme` in `layout.css` (colors: `pink`, `pink-highlight`, `black`, `gray`, `white`, `globe-default`; custom font: Nohemi; custom utilities: `pb-safe`, `pb-nav-safe`)
- **Formatting**: Tabs, single quotes, no trailing commas, printWidth 100
- **TypeScript**: Strict mode enabled

## MCP Tools (Svelte Documentation)

You have access to a Svelte MCP server. Use these tools when working on Svelte/SvelteKit code:

1. **list-sections** — Call FIRST to discover available documentation sections. Analyze the `use_cases` field to find relevant docs.
2. **get-documentation** — Fetch full documentation for relevant sections found via list-sections.
3. **svelte-autofixer** — MUST be used whenever writing Svelte code. Keep calling until no issues remain.
4. **playground-link** — Generates a Svelte Playground link. Only offer after code is complete, and NEVER if code was written to project files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nabal22)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nabal22)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
