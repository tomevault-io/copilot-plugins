## ogv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static website for OGV Pattenhofen-Altenthann e.V. (a German fruit and garden association), built with Astro 5.0. The site is fully static with no CMS — all content is hardcoded in Astro components. All page content is in German.

**Live site:** https://www.ogv-altenthann-pattenhofen.de

## Commands

| Command | Purpose |
|---------|---------|
| `npm run dev` | Start development server |
| `npm run build` | TypeScript check + production build |
| `npm run typecheck` | TypeScript validation only (`astro check`) |
| `npm run format` | Format code with Prettier |
| `npm run preview` | Preview production build locally |

There are no tests configured.

## Architecture

**Framework:** Astro 5.0 with file-based routing. Zero client-side JavaScript shipped.

**Page pattern:** Every page in `src/pages/` follows the same structure — import `Layout.astro`, pass props (`title`, `teaserImage`, `teaserPosition`, `teaserAltText`), and provide HTML content in the slot.

**Layout:** Single shared layout (`src/layouts/Layout.astro`) handles header, responsive hamburger navigation (CSS-only checkbox toggle), footer, and meta tags. Navigation links are hardcoded in the layout.

**Styling:** SCSS with a Minima-inspired theme system:
- `src/styles/minima/_variables.scss` — design tokens (colors, breakpoints, spacing)
- `src/styles/minima/_base.scss`, `_layout.scss`, `_custom.scss` — global styles
- Breakpoints: `$on-palm` (600px), `$on-laptop` (800px)
- Fonts: Crete Round (body), Patua One (headings), self-hosted in `src/styles/fonts/`

**TypeScript:** Path aliases configured in `tsconfig.json`: `@/*` → `src/*`, plus `@components/*`, `@layouts/*`, `@styles/*`.

## Deployment

Push to `master` triggers GitHub Actions → `npm run build` → FTP deploy of `dist/` to the production server. PR builds run typecheck + build as validation. FTP credentials are in GitHub secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markuseckstein)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/markuseckstein)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
