## webapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nepali Society - Fargo Moorhead community website. A Nuxt 4 static site with Supabase backend, deployed to GitHub Pages.

## Development Commands

```bash
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm generate     # Generate static site for GitHub Pages
pnpm preview      # Preview production build
```

## Architecture

**Stack**: Nuxt 4 (Vue 3) + TypeScript + Supabase + FontAwesome

**Key patterns**:
- SSR disabled (`ssr: false`) - client-side rendering only, static site generation
- Supabase client injected globally via plugin as `$supabase`
- Composables in `app/composables/` for data fetching (useEvents, useMembers, useCommittee, useSupabaseImage)
- File-based routing in `app/pages/`
- Single default layout wrapping all pages

**Data flow**: Pages/components → composables → Supabase queries → reactive data

**Image handling**: `useSupabaseImage` composable generates URLs for Supabase Storage buckets (public and signed private URLs)

## Environment Variables

Required in `.env`:
- `NUXT_PUBLIC_SUPABASE_URL`
- `NUXT_PUBLIC_SUPABASE_ANON_KEY`

## Deployment

GitHub Actions workflow (`.github/workflows/front.yaml`) builds and deploys to GitHub Pages on push to main. Uses Nitro's `github_pages` preset.

## Code Style

Prettier configured: 100 char width, single quotes, 2-space tabs.

---
> Source: [fmnepalisociety-dev/webapp](https://github.com/fmnepalisociety-dev/webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
