## tfm-client

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tokyo FM Podcast Client — a full-stack Nuxt 4 app that scrapes Tokyo FM's podcast website using Puppeteer and serves a Vue 3 SPA for browsing programs and playing episodes.

## Commands

- **Install**: `bun install`
- **Dev server**: `bun run dev` (http://localhost:3000)
- **Build + preview**: `bun run start`
- **Lint**: `bun run lint` / `bun run lint:fix`
- **Test all**: `bun test`
- **Single test**: `bun test server/scraping/programs.test.ts`

Git hooks (lefthook): pre-commit runs `lint:fix`, pre-push runs `bun test`.

## Architecture

**Runtime**: Bun + Nuxt 4 (Vue 3 + Nitro server)

### Frontend (`app/`)
- Nuxt 4 directory layout — pages, components, layouts live under `app/`
- File-based routing: `/` (program search) → `/:program/episodes` (episode list)
- UI: Nuxt UI v4 components (`UApp`, `UContainer`, `UInput`, `UCard`, `UButton`, `UProgress`, `useToast`) + Lucide icons
- Data fetching via `useFetch()` with `immediate: true`

### Backend (`server/`)
- Nitro auto-routes under `server/api/`
- API: `GET /api/programs?name=<query>`, `GET /api/:program/episodes`
- Query/route param validation with Zod schemas via `getValidatedQuery` / `getValidatedRouterParams`
- Scraping logic in `server/scraping/` — Puppeteer extracts data from TFM website
- RSS feed discovery and parsing in `server/scraping/rss.ts`

### Shared types (`shared/model/`)
- `Program` (id, title, href, img) and `Episode` (id, title, description, durationSeconds, publishedAt, audio, thumbnail)
- Used by both frontend and backend via Nuxt auto-imports

### Key pattern: Puppeteer singleton
`server/utils/browser.ts` manages a single browser instance. `withBrowser(callback)` creates a page, runs the callback, and closes the page.

## Code Style

- TypeScript, 2-space indent, single quotes, no semicolons
- ESLint with `@nuxt/eslint` stylistic rules, 1TBS brace style — no Prettier
- Vue SFC order: `<template>` first, then `<script setup lang="ts">` — no `<style>` blocks (use Tailwind classes)
- Use Composition API with `<script setup>`, `defineProps<{...}>()` for typed props
- Import aliases: `~/` for `app/` directory, `~~/` for project root (e.g., `~~/shared/model/program`)
- Use `interface` (not `type`) for shared model definitions
- Tests use `bun:test` with long timeouts (20-30s) since they hit the real TFM website

## Nuxt UI MCP

A Nuxt UI MCP server is configured in `.mcp.json` for looking up Nuxt UI v4 component APIs. Use it when working with UI components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/y-u-t-a) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
