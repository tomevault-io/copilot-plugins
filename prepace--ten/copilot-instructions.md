## ten

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Quick Start Commands

```bash
npm run dev       # Start Next.js development server on http://localhost:3000
npm run build     # Build for production
npm start         # Start production server
npm run lint      # Run ESLint checks
```

## Project Overview

This is a Next.js 15 fundraising website for the **Tribal Energy Network — CALeVIP Fundraiser**, a campaign to help 10 California Tribes secure clean energy infrastructure.

**Project Type:** Single-page marketing/fundraising website
**Tech Stack:** Next.js 15, React 19, Tailwind CSS 4, JavaScript (with TypeScript configuration)

## Architecture

### Page Structure

The application uses Next.js App Router with the following pages:

1. **Home Page** (`/app/page.jsx`)
   - Tribe selector landing page with CA Tribes and WA Tribes buttons
   - Uses client-side state management with React hooks (useState)
   - Merges CA and WA content templates with conditional rendering based on selected tribe
   - Marked as client component with `"use client"` directive

2. **Investor Page** (`/app/support/page.jsx`)
   - Original home page content (fundraiser focused)
   - Contains: Investment tiers, KPIs, FAQ, timeline, budget breakdown
   - Target audience: Private sponsors and institutional investors
   - Goal: $500,000 fundraise with tiered participation levels

### Styling & Theme

- **Framework:** Tailwind CSS 4 with custom CSS variables
- **Global Styles:** `/app/globals.css` defines CSS custom properties (--bg, --fg, --muted, --primary, etc.)
- **Theme Config:** `/tailwind.config.js` - Customize colors and Tailwind configuration
- **PostCSS:** Configured with Tailwind and autoprefixer plugins

**Key Style Approach:** Utility-first CSS via Tailwind; no CSS-in-JS libraries

### Component Organization

- Components are defined inline within page files (no separate component directory)
- Uses React hooks for interactivity (useState for form/UI state, useEffect for side effects)
- Utility components included: StatCard, Countdown, and other visual elements
- Next.js Image component used for asset optimization

### Root Configuration

- **Layout:** `/app/layout.jsx` - Sets up root HTML, metadata, and applies global styles
- **Public Assets:** `/public/` contains static images (SVGs and JPGs: ten0.jpg through ten9.png)

## Technologies & Key Configurations

- **TypeScript:** Configured via `tsconfig.json` but project is written in JavaScript/JSX
- **Module Aliases:** Set in `jsconfig.json` for cleaner imports
- **Linting:** ESLint with Next.js core-web-vitals configuration (see `eslint.config.mjs`)
- **Formatting:** Biome configured but ESLint is the active linter

## Environment Variables

Create `.env.local` in the project root with:

```
NEXT_PUBLIC_BOOKING_URL=https://calendly.com/ampace
OPENAI_API_KEY=sk-proj-...
```

- `NEXT_PUBLIC_BOOKING_URL` - Used for Calendly integration on booking buttons
- `OPENAI_API_KEY` - Available for API calls (used for unknown functionality currently)

## Key Files to Understand

| File | Purpose |
|------|---------|
| `/app/page.jsx` | Tribe selector home page with CA/WA toggle (uses useState) |
| `/app/ca/page.jsx` | CA tribes offer details with navigation selector |
| `/app/wa/page.jsx` | WA tribes offer details with navigation selector |
| `/app/support/page.jsx` | Investor fundraising page (moved from original home) |
| `/app/layout.jsx` | Root layout and metadata |
| `/app/globals.css` | Global styles and CSS variable definitions |
| `/tailwind.config.js` | Tailwind theme and content configuration |
| `/package.json` | Dependencies and npm scripts |
| `jsconfig.json` | Module path aliases |
| `next.config.mjs` | Next.js config (React Strict Mode enabled) |

## Routing Structure

- **`/`** - Tribe selector home page with CA Tribes and WA Tribes buttons
  - Uses client-side state to toggle between CA and WA content
  - Single page component with CAContent and WAContent sub-components
  - Includes direct navigation link to `/support` for investors

- **`/ca`** - California tribes-focused offer page
  - Displays CALeVIP FCCP program details
  - Includes tribe selector buttons at top for switching to WA or investor pages
  - Navigates to `/wa` and `/support` via buttons

- **`/wa`** - Washington tribes-focused offer page
  - Displays WAEVCP2 program details
  - Includes tribe selector buttons at top for switching to CA or investor pages
  - Navigates to `/ca` and `/support` via buttons

- **`/support`** - Investor-focused fundraising page
  - Original home page content
  - Contains investment tiers, KPIs, FAQ, timeline
  - Accessible from home page and regional pages via "For Investors" button

## Development Notes

- **No External State Management:** Uses React hooks directly; no Redux, Zustand, or similar
- **No API Routes:** Current pages are static/client-rendered
- **TypeScript vs JavaScript:** TypeScript is configured but not actively used; codebase is JavaScript
- **Performance:** Uses Next.js Image component and optimized CSS loading
- **Client-Side Rendering:** Pages use `"use client"` and manage state with React hooks
- **Tribe Selector Pattern:** Home page uses conditional rendering for CA/WA content; regional pages use Next.js Link for navigation

## Git & Recent Changes

- Repository is clean with recent commits: `standard`, `contact`, `margin`, `ca`, `offers`
- Main branch is the primary branch for development

---
> Source: [prepace/ten](https://github.com/prepace/ten) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
