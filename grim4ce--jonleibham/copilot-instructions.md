## jonleibham

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for Jon Leibham, built with React 19, TypeScript, and Vite. Deployed via AWS Amplify.

## Commands

- `npm run dev` — Start dev server (localhost:5173)
- `npm run build` — TypeScript check + Vite production build (outputs to `dist/`)
- `npm run lint` — ESLint across the project
- `npm run preview` — Preview the production build locally

Node version 24.12.0 is pinned via Volta.

## Architecture

**Component pattern:** Each component lives in `src/components/<Name>/` with three files:
- `<Name>.tsx` — React functional component
- `<Name>.module.css` — CSS Modules scoped styles
- `index.ts` — Barrel re-export

**Styling:** CSS Modules for component scoping, global design tokens (colors, typography) defined in `src/index.css`. No CSS framework — vanilla CSS only.

**No backend:** The contact form uses client-side `mailto:` with email obfuscation (split into parts at runtime) and a honeypot field for spam protection.

**Deployment:** AWS Amplify configured in `amplify.yml` — builds with `npm run build`, serves from `dist/`, includes security headers and immutable asset caching.

## Key Files

- `src/index.css` — Color palette and global CSS variables
- `index.html` — SEO meta tags, Open Graph, Twitter cards, favicon/PWA config
- `amplify.yml` — AWS Amplify build pipeline and custom headers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GRIM4CE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
