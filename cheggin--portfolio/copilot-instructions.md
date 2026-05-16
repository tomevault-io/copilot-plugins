## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A portfolio website repository containing:
- **Portfolio-Latest/**: The current minimalist portfolio (React, TypeScript, Vite, Tailwind)
- **V1/**: Previous "Liquid Glass" portfolio with advanced animations, fluid cursor effects, and WebGL (archived and served at `/v1/`)

The current portfolio is a clean, readable design focusing on content. The V1 archive showcases the previous interactive design with glassmorphism UI and advanced visual effects. Both are deployed together on Vercel.

## Development Commands

### Current Portfolio (Portfolio-Latest/)

**Working Directory**: All development commands should be run from the `Portfolio-Latest/` subdirectory.

```bash
cd Portfolio-Latest/

# Development
npm run dev          # Start Vite dev server

# Building
npm run build        # TypeScript compile + Vite build

# Linting
npm run lint         # Run ESLint

# Preview
npm run preview      # Preview production build
```

### V1 Archive Portfolio

The V1 portfolio is a built React app served from `Portfolio-Latest/public/v1/`. It's accessible at `/v1/` when the main site is deployed.

**To update the V1 archive:**

```bash
# 1. Navigate to V1 source directory
cd V1/

# 2. Make your changes to the source files

# 3. Build the V1 portfolio (skips TypeScript errors, produces static files)
npx vite build

# 4. Copy built files to the main portfolio's public directory
rm -rf ../Portfolio-Latest/public/v1
cp -r dist ../Portfolio-Latest/public/v1
```

**Important notes:**
- V1 uses `basename="/v1"` in BrowserRouter to work correctly when served from `/v1/`
- V1 uses `base: './'` in vite.config.ts for relative asset paths
- V1 has its own `.env` file with Supabase credentials (same as Portfolio-Latest)
- The built V1 files are checked into git in `Portfolio-Latest/public/v1/`

## Architecture

### Routing & Pages
- **React Router** for client-side routing with SPA architecture
- Main routes defined in [App.tsx](Portfolio/src/App.tsx):
  - `/` - Home page with hero and call-to-action buttons
  - `/about` - About page with background and interests
  - `/projects` - Projects showcase
  - `/resume` - Resume and experience
  - `/contact` - General contact form
  - `/recruiter` - Dedicated recruiter landing page
  - `/recruiter-contact` - Recruiter-specific contact

- **Layout Component**: Wraps all routes with `UniversalNavbar` and manages active navigation state
- **Intro Animation**: First-time session experience using `sessionStorage` to show only once per session

### UI Component System
All custom components are in [src/components/](Portfolio/src/components/). The design system is built around a "liquid glass" aesthetic:

- **LiquidGlassCard**, **LiquidGlassButton**, **LiquidGlassModal**, **LiquidGlassNav** - Core glassmorphism components with blur effects
- **FluidCursor** / **FluidCursorEffect** - WebGL-based fluid simulation that follows mouse movement
- **AppleGlowName** - Hero text with Apple-style glow effect
- **IntroAnimation** - Full-screen intro sequence with HyperSpeed starfield effect
- **TiltedCard**, **MetaBalls** - Advanced visual effect components
- **VisitorCounter** - Tracks site visits using Supabase

### Data & Backend
- **Supabase Integration**: Client configured in [supabaseClient.ts](Portfolio/src/supabaseClient.ts)
  - Environment variables: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`
  - Used for visitor counting and contact form submissions
- **Project Data**: Static project information in [projectsData.ts](Portfolio/src/components/projectsData.ts)

### Animation Libraries
- **framer-motion**: Page transitions and component animations
- **gsap**: Timeline-based animations
- **three.js + postprocessing**: 3D graphics and WebGL effects
- **matter-js**: Physics simulations
- **ogl**: Lightweight WebGL framework for fluid simulation

### Styling
- Custom CSS with glassmorphism effects
- Responsive design with mobile breakpoints (typically 700px)
- Global styles in `index.css` and component styles in `App.css`

## Configuration Files

- **vite.config.ts**: Vite configuration with React plugin
- **tsconfig.json**: TypeScript compiler configuration (references `tsconfig.app.json` and `tsconfig.node.json`)
- **eslint.config.js**: ESLint setup with React and TypeScript rules
- **vercel.json**: Vercel deployment config with SPA rewrites

## Environment Variables

Store in `Portfolio/.env`:
- `VITE_SUPABASE_URL` - Supabase project URL
- `VITE_SUPABASE_ANON_KEY` - Supabase anonymous key

## Key Patterns

### Navigation Flow
The `navConfig.ts` exports both `navItems` and a `handleNavItemClick` function that manages navigation state and React Router navigation together. This ensures the active nav item stays in sync with the route.

### Session Management
The intro animation uses `sessionStorage.getItem('introShown')` to detect if the user has seen the intro this session. This provides a smoother experience for returning visitors without persistence across browser sessions.

### Fluid Effects
The fluid cursor effect is implemented using WebGL shaders and runs as a background effect that responds to mouse movement. The simulation state is managed in `FluidSimulation.ts` with React components wrapping the WebGL canvas.

---
> Source: [Cheggin/Portfolio](https://github.com/Cheggin/Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
