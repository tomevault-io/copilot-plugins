## places

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Plaziia** is a commercial property registration and analysis platform that allows property owners to:
- Register commercial properties with detailed metadata (location, size, price, amenities)
- View competitors within a 2km radius (Pro plan)
- Analyze consumption trends and get AI-powered business recommendations (Pro plan)
- Download professional property reports (Pro plan)

The app features a modern dark theme UI with gradient accents.

## Tech Stack

- **Framework**: Next.js 16.1.6 (App Router)
- **Language**: TypeScript
- **Auth**: NextAuth v5.0.0-beta with Google OAuth
- **UI Libraries**: Material-UI (MUI) 5.18.0, Emotion (styled components)
- **Forms**: React Hook Form 7.71.2
- **Maps**: Leaflet + React Leaflet (dynamic import, SSR disabled)
- **Styling**: Emotion + Tailwind-style CSS variables for theming

## Essential Commands

```bash
# Install dependencies
npm install

# Development server (hot reload)
npm run dev

# Build for production
npm build build

# Start production server
npm start

# Lint code
npm run lint
```

## Architecture & Key Files

## Code style

- User server actions where is possible
- Add code in English
- Respect eslint rules


### Authentication & Middleware
- **`auth.ts`**: NextAuth configuration with Google provider. Exports `handlers`, `auth`, `signIn`, `signOut`
- **`middleware.ts`**: Protects `/registro` route - redirects unauthenticated users to `/login`
- **`app/api/auth/[...nextauth]/route.ts`**: Exposes NextAuth handlers as API route

**Flow**: Unauthenticated → `/login` (Google OAuth) → redirect to `/registro` → can register property

### Pages & Routes
- **`app/page.tsx`**: Landing page - hero section, features, pricing, how-it-works
- **`app/login/page.tsx`**: Google OAuth login button
- **`app/registro/page.tsx`** (not shown): Registration page (protected route, likely imports `Plaziia` component)
- **`app/propiedades/page.tsx`** (not shown): Properties listing page

### Components
- **`components/Plaziia.tsx`**: Multi-step registration form (3 steps: Location → Features → Details)
  - Uses MUI Stepper + React Hook Form
  - Includes location picker via Leaflet (MapPicker component)
  - Form fields: neighborhood, location, property type, size, utilities, rooms/baths/parking, price, maintenance
  - **TODO**: Backend persistence not implemented (line 176: `// TODO: persist to backend/database`)
  
- **`components/MapPicker.tsx`** (not shown): Dynamic Leaflet map for location selection
- **`components/Providers.tsx`**: Client-side provider wrapper with NextAuth SessionProvider
- **`components/NavHeader.tsx`** (not shown): Navigation header

### Styling & Theme
- **`app/globals.css`**: Global styles and CSS variables
- **Theme colors**: 
  - Primary accent: `#00f5a0` (green)
  - Secondary: `#00b4d8` (cyan)
  - Backgrounds: `#0a0a14`, `#0e0e22` (dark)
  - Text: `#e0e0ff` (light)
- MUI theme uses dark palette with both fonts ("Syne" for headings, "DM Mono" for body/code)

### Configuration
- **`next.config.mjs`**: Minimal config with `reactStrictMode: true`
- **`tsconfig.json`**: Path alias `@/*` maps to root directory
- **`.env.local`**: Must contain `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`

## Development Patterns

### Form Handling
- Use React Hook Form with Controller pattern (see Plaziia.tsx ~140-400)
- Validation: `rules` prop on Controller (required, min/max, custom)
- Multi-step forms: trigger validation per step before advancing

### Styling Approach
- MUI components + Emotion theming
- Inline styles for layout-critical elements (flexbox, positioning)
- CSS classes for noise overlays and gradients (.gradient-text, .tag, .card, .btn-primary, .btn-secondary)
- Avoid adding new component libraries - expand existing MUI theming

### Maps Integration
- Leaflet imported dynamically with `next/dynamic` (SSR disabled)
- Only `MapPicker` component handles map logic (avoid inline map code elsewhere)

### Dark Theme
- All new components should respect CSS variables (--accent, --background, --surface, --muted, etc.)
- Check `globals.css` for the full variable list before hardcoding colors

## Known Issues & TODOs

1. **Backend Integration**: Plaziia form submission is stubbed (line 176-178) - needs backend API to persist registrations
2. **MapPicker Component**: Referenced but implementation not shown (in `app/components/MapPicker.tsx`)
3. **Registro Page**: Imports Plaziia but implementation not visible

## Testing & Deployment Notes

- No tests currently configured (no jest/vitest setup)
- Production build requires valid Google OAuth credentials in environment
- Auth callback redirects to `/registro` after login (could be configurable)

---
> Source: [kinglee18/places](https://github.com/kinglee18/places) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
