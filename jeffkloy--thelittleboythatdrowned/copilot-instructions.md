## thelittleboythatdrowned

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a poetry website built with React 19 + Vite + TypeScript that displays poems with their analyses. The site supports:
- Dynamic poem loading from markdown files
- Tag-based filtering
- Analysis viewing with a toggle interface
- Responsive design optimized for mobile and desktop
- GitHub Pages deployment with SPA routing

## Tech Stack

- React 19.1 (function components + hooks)
- Vite (development and build tooling)
- TypeScript (configured with project references)
- React Router (BrowserRouter with basename for GitHub Pages)
- No CSS framework - custom styles in `src/styles/globals.css`

## Directory Structure

- `public/poems/` - Contains all poem markdown files and `poems.json` metadata
- `public/analyses/` - Contains analysis markdown files for each poem
- `src/components/` - React components (Header, Particles, TagFilters, PoemList, PoemView)
- `src/hooks/` - Custom hooks (usePoems, usePoem)
- `src/lib/` - Utilities (markdown parser)

## Key Development Commands

```bash
# Install dependencies
npm install

# Start development server (runs on http://localhost:5173/thelittleboythatdrowned/)
npm run dev

# Build for production
npm run build

# Preview production build locally
npm run preview

# Run linting (ESLint configured)
npm run lint

# Format code (Prettier configured)
npm run format

# Type checking
npm run typecheck
```

## Custom Commands

- `/sync-poems` - Synchronize poetry collection with automatic tag extraction and AI analysis generation (see `.claude/commands/sync-poems.md`)

## Architecture

### Data Flow
1. `poems.json` contains metadata for all poems (filename + tags)
2. `usePoems` hook fetches and parses poems.json on app load
3. Individual poems/analyses are loaded on-demand when selected
4. Tag filtering happens client-side based on poem metadata

### Component Architecture
- `App.tsx` - Main layout orchestrator, manages selected poem state
- `NavContent` - Shared navigation content for desktop sidebar and mobile drawer
- `Header` - Site header with mobile menu support (accessibility-focused)
- `TagFilters` - Tag selection UI with counts
- `PoemList` - Filtered poem list display
- `PoemView` - Individual poem display with analysis toggle

### Routing
- Single-page application with client-side routing
- Base path configured as `/thelittleboythatdrowned/` for GitHub Pages
- 404.html fallback ensures SPA routing works on GitHub Pages

### State Management
- React hooks for local state (useState, useEffect, useMemo)
- No external state management library
- Poem selection and tag filtering managed in App component

## Deployment

- GitHub Actions workflow (`.github/workflows/deploy.yml`) handles automated deployment
- Builds on push to `main` branch
- Deploys to `gh-pages` branch
- Base path must match repository name for GitHub Pages

## Performance Optimizations

- Poems list prefetched at startup
- Next poem prefetched on selection for faster navigation
- Individual poems/analyses loaded on-demand
- Mobile menu closes automatically on poem selection

## Accessibility Features

- Semantic HTML structure with proper ARIA attributes
- Mobile menu with keyboard navigation (ESC to close)
- Focus management on menu open/close
- Screen reader support with aria-expanded and aria-controls
- Scroll lock when mobile menu is open

---
> Source: [jeffkloy/thelittleboythatdrowned](https://github.com/jeffkloy/thelittleboythatdrowned) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
