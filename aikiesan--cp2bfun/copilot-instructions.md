## cp2bfun

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CP2B Website - Institutional website for Centro Paulista de Estudos em Biogas e Bioprodutos (Sao Paulo Center for Biogas and Bioproducts Studies). A bilingual (Portuguese/English) React SPA hosted on Vercel.

## Commands

All commands must be run from the `cp2b_web/` directory:

```bash
cd cp2b_web

# Development
npm run dev          # Start Vite dev server (http://localhost:5173)

# Build
npm run build        # Production build to dist/

# Lint
npm run lint         # ESLint check (JS/JSX, zero warnings policy)

# Preview production build
npm run preview      # Preview built app locally
```

## Docker

Docker configuration is available for local development and production deployment (Debian 12 VM):

```bash
cd cp2b_web

# Development mode (hot reload on http://localhost:5173)
docker-compose up dev

# Production mode (optimized build on http://localhost:80)
docker-compose up prod

# Build production image only
docker-compose build prod

# Run in background (for VM deployment)
docker-compose up -d prod

# Stop containers
docker-compose down
```

## Architecture

### Tech Stack
- React 18 + Vite 5
- React Router v6 (client-side routing)
- React Bootstrap + Bootstrap 5 (UI framework)
- Framer Motion (animations)
- Deployment: Vercel with SPA rewrite rules

### Source Structure (`cp2b_web/src/`)

```
src/
├── App.jsx              # Root: Router setup, LanguageProvider, layout (Header/Footer)
├── main.jsx             # React entry point
├── index.css            # Global styles
├── components/          # Shared components (Header, Footer, ScrollToTop)
├── context/
│   └── LanguageContext.jsx   # i18n state (pt/en), persisted to localStorage
├── data/
│   └── content.js       # All translatable content and static data
└── pages/               # Route pages (Home, About, Research, Team, News, etc.)
```

### Internationalization Pattern

All text content is centralized in `src/data/content.js` with `pt` and `en` keys. Pages consume translations via the `useLanguage()` hook:

```jsx
import { useLanguage } from '../context/LanguageContext';
import { menuLabels } from '../data/content';

const { language } = useLanguage();
const t = menuLabels[language];  // Access translated strings
```

When adding new text, add both Portuguese and English versions to the relevant export in `content.js`.

### Routing

Routes use Portuguese paths (`/sobre`, `/pesquisa`, `/equipe`, etc.). The Vercel config (`vercel.json`) rewrites all paths to `index.html` for SPA client-side routing.

### Static Assets

Images and media are in `cp2b_web/public/assets/` and referenced as `/assets/filename.ext` in code.

---
> Source: [aikiesan/cp2bfun](https://github.com/aikiesan/cp2bfun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
