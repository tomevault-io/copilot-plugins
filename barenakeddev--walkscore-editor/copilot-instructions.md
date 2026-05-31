## walkscore-editor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A React-based infographic editor for creating Walk Score marketing materials for hotels. Users can edit text inline, upload hotel images, and export the entire infographic as a PNG file for social media sharing.

## Development Commands

**Install dependencies:**
```bash
pnpm install
```

**Development server:**
```bash
pnpm dev
# Runs Vite dev server on http://localhost:3000 (or next available port)
# Server accessible via --host flag for network access
```

**Build:**
```bash
pnpm build
# Builds both client (Vite) and server (esbuild) to dist/
# Client output: dist/public/
# Server output: dist/index.js
```

**Production server:**
```bash
pnpm start
# Runs production build from dist/index.js
# Requires build to be run first
```

**Type checking:**
```bash
pnpm check
# Runs TypeScript compiler without emitting files
```

**Format code:**
```bash
pnpm format
# Runs Prettier on all files
```

## Architecture

### Monorepo Structure

**Three-directory monorepo:**
- `client/` - React SPA with Vite build system
- `server/` - Express server that serves static files and handles client-side routing
- `shared/` - Shared TypeScript constants and types

**Build outputs:**
- Client builds to `dist/public/`
- Server builds to `dist/index.js` (bundled with esbuild)
- Production server serves from `dist/public/` and falls back to `index.html` for all routes

### Path Aliases

Configured in both `tsconfig.json` and `vite.config.ts`:
- `@/*` → `client/src/*`
- `@shared/*` → `shared/*`
- `@assets/*` → `attached_assets/*`

### Tech Stack

**Frontend:**
- React 18 with TypeScript
- Vite for build/dev server
- Wouter for client-side routing (patched version in `patches/`)
- TailwindCSS 4.x with `@tailwindcss/vite` plugin
- shadcn/ui components (New York style, neutral base color)
- Radix UI primitives
- html2canvas for PNG export functionality

**State Management:**
- React hooks for local state
- React Context for theme management (`ThemeContext`)

**Server:**
- Express.js with minimal configuration
- SPA fallback routing (all routes → `index.html`)
- Static file serving from `dist/public/`

### Application Flow

1. Entry point: `client/src/main.tsx` renders `App`
2. `App.tsx` sets up global providers:
   - `ErrorBoundary` for error handling
   - `ThemeProvider` (default: light, switchable disabled by default)
   - `TooltipProvider` from Radix UI
   - `Router` using Wouter
3. Routes defined in `App.tsx`:
   - `/` → `Home` component
   - `/404` and fallback → `NotFound` component
4. `Home` renders `InfographicEditor` component (main application)

### InfographicEditor Component

The core component at `client/src/components/InfographicEditor.tsx`:

**Key features:**
- All text elements use `contentEditable` for inline editing
- Image upload via file input (stored as data URL in state)
- PNG export using html2canvas with:
  - 2x scale for high resolution (1960×1544px)
  - White background
  - OKLCH to RGB color conversion in cloned DOM (for browser compatibility)
  - Data attributes for element tracking during export

**Data flow:**
- Image upload → FileReader → data URL → state → img src
- Export → html2canvas clone → color conversion → canvas → blob → download

### UI Component System

**shadcn/ui configuration** (`components.json`):
- Style: `new-york`
- Base color: `neutral`
- CSS variables enabled
- Components located in `@/components/ui/`

**Custom components:**
- `ErrorBoundary` - React error boundary with fallback UI
- `ManusDialog` - Custom dialog component
- `InfographicEditor` - Main application component

### Custom Hooks

Located in `client/src/hooks/`:
- `useComposition.ts` - Composition event handling utilities
- `useMobile.tsx` - Mobile device detection
- `usePersistFn.ts` - Function reference persistence

### Styling Approach

- TailwindCSS 4.x for utility classes
- CSS variables for theme colors (defined in `client/src/index.css`)
- Inline styles for specific color values in infographic (e.g., `#3d4a5c`, `#5a6a7d`)
- Theme switcher commented out by default in `ThemeProvider`

### Static Assets

Public assets in `client/public/`:
- Icon images for infographic sections (food-beverage, parks-museums, etc.)
- Transportation mode icons (bus, car)
- Venue logos (Rogers Centre, Rideau Centre)

**Important:** Asset paths in production reference `/icon-name.png` (served from `dist/public/`)

### Package Management

**pnpm with patches:**
- `wouter@3.7.1` has a custom patch in `patches/wouter@3.7.1.patch`
- Version override: `tailwindcss>nanoid` locked to `3.3.7`
- Package manager: `pnpm@10.4.1`

### TypeScript Configuration

- Strict mode enabled
- Module resolution: `bundler` (Vite-compatible)
- Import TypeScript extensions allowed (`.ts`, `.tsx`)
- `noEmit: true` (Vite handles compilation)
- Incremental compilation with build info cache

### Server Configuration

**Vite dev server** (`vite.config.ts`):
- Port 3000 with fallback to next available
- Allowed hosts configured for Manus cloud environments
- Strict file serving with dotfile deny rules
- Root set to `client/` directory

**Production server** (`server/index.ts`):
- Minimal Express setup
- Static file serving with SPA fallback
- Port from `PORT` env var or 3000
- ESM module format

### Build System

**Dual build process:**
1. Vite builds client SPA → `dist/public/`
2. esbuild bundles server → `dist/index.js` (ESM format, external packages)

**Build considerations:**
- Client uses Vite plugins: React, TailwindCSS, JSX location tracking, Manus runtime
- Server is bundled as single file with platform: node
- Production build empties `dist/public/` before building

---
> Source: [barenakeddev/walkscore-editor](https://github.com/barenakeddev/walkscore-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
