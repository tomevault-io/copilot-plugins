## frekvens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Frekvens is a bilingual (Norwegian/English) single-page website for an Oslo underground DJ collective. The site features custom WebGL/Canvas visual effects and data-driven content for team members and artists.

## Development Commands

```bash
# Development server
npm run dev

# Build for production (runs TypeScript compiler + Vite build)
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Tech Stack

- **Framework**: React 18 + TypeScript
- **Build Tool**: Vite
- **Styling**: Tailwind CSS with custom theme (primary red: #ff3131, dark palette)
- **3D/Visual Effects**:
  - Three.js + @react-three/fiber + @react-three/postprocessing
  - OGL (custom WebGL shaders)
  - Custom Canvas 2D effects
- **Animation**: Framer Motion
- **Icons**: lucide-react

## Architecture

### Application Structure

The app is a single-page application with the following sections (in order):
1. Hero - Landing section with visual effects
2. Mission - About/mission statement
3. People - Team members section
4. Roster - Artists/DJs roster
5. Contact - Contact information and social links

### Component Organization

- `src/components/sections/` - Main page sections (Hero, Mission, People, Roster, Contact)
- `src/components/layout/` - Layout components (Header, Footer)
- `src/components/ui/` - Reusable UI components and effects
  - `BitsDither.tsx` - Canvas-based Bayer dithering background effect
  - `Threads.tsx` - WebGL shader effect using OGL (animated wavy lines with Perlin noise)
  - `PersonCard.tsx` - Card component for displaying people/artists
  - `LanguageToggle.tsx` - Language switcher component

### Data Flow

**Content is data-driven** - People and Roster sections load data from JSON files:
- `public/data/people.json` - Team members data
- `public/data/roster.json` - Artists/DJs data
- Data fetched via custom hooks: `usePeopleData()`, `useRosterData()`
- Images stored in `public/images/people/` and `public/images/roster/`

**See `public/data/README.md` for the data schema and how to add new entries.**

### Internationalization (i18n)

The site supports Norwegian (no) and English (en):
- `src/context/LanguageContext.tsx` - Language state management with localStorage persistence
- `src/data/translations.ts` - UI string translations
- Language auto-detected from browser, persisted to localStorage
- Access via `useLanguage()` hook: `const { language, setLanguage, t } = useLanguage()`
- Person/roster data includes bilingual fields: `role: { no: string, en: string }`

### Visual Effects

**BitsDither** (`src/components/ui/BitsDither.tsx`):
- Canvas 2D-based dithering effect using Bayer 8×8 matrix
- Props: `colors` (color palette), `intensity` (effect strength)
- Used as background overlay

**Threads** (`src/components/ui/Threads.tsx`):
- WebGL shader effect using OGL library
- Creates animated wavy lines with Perlin noise
- Props: `color` (RGB array), `amplitude`, `distance`, `enableMouseInteraction`
- Uses custom vertex/fragment shaders defined inline

## Styling

Tailwind is configured with a custom theme ([tailwind.config.js](tailwind.config.js)):
- **Primary color**: `#ff3131` (red)
- **Dark palette**: `dark-900` (#0a0a0a), `dark-800` (#121212), `dark-700` (#1a1a1a), `dark-600` (#242424)
- **Custom fonts**: Lastica (display), Inter (sans-serif)
- **Custom effects**: Glow shadows, pulse animations, radial gradients

Global styles in [src/styles/globals.css](src/styles/globals.css).

## Type Definitions

Core types in [src/types/index.ts](src/types/index.ts):
- `Language` - 'no' | 'en'
- `Person` - Schema for people/roster entries (includes bilingual role/description, imageUrl, links)
- `TranslationStrings` - UI translation structure
- `Translations` - Map of Language to TranslationStrings

## Scripts

- `scripts/fetch-instagram-profiles.js` - Utility for fetching Instagram profile data

## GitHub Pages Deployment

The site is configured for GitHub Pages deployment (see `public/CNAME`). Videos use MP4 format for GitHub Pages compatibility.

---
> Source: [andreasklae/Frekvens](https://github.com/andreasklae/Frekvens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
