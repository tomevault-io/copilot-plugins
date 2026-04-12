## web-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev       # Start development server at localhost:4321
pnpm build     # Build for production (output: dist/)
pnpm preview   # Preview production build
```

## Architecture

This is a personal portfolio built with **Astro 5** using static site generation.

### Key Files

- `astro.config.mjs` - Astro config with MDX and sitemap integrations, Shiki syntax highlighting
- `src/layouts/Layout.astro` - Main layout with navigation, footer, theme toggle, and perspective switcher
- `src/styles/global.css` - CSS variables for theming (light/dark mode), color system uses `--color-yellow: #ffdd00` as accent
- `src/content/config.ts` - Defines blog collection schema with Zod

### Content System

Blog posts use Astro's content collections stored in `src/content/blog/` as MDX files. The schema requires:
- `title`, `description`, `pubDate` (required)
- `tags`, `draft`, `updatedDate` (optional)

### Perspective Switcher

The layout includes a floating perspective switcher (backend/making/activism) that filters content using `data-perspectives` attributes on elements. State persists in localStorage.

### Theming

Dark/light theme controlled via `data-theme="dark"` attribute on `<html>`. Toggle state persists in localStorage.

## TypeScript

Uses Astro's strict tsconfig. Avoid `any` type - use `unknown` for uncertain data structures.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dissfall)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dissfall)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
