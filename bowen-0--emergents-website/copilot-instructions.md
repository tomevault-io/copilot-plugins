## emergents-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview
```

## Deployment

Hosted on Cloudflare Pages (project `emergents-website`, custom domain **emergents.io**).
This is a **Direct-Upload** project — **not** git-connected, so `git push` does **not** deploy.
Publishing is manual:

    npm run build
    wrangler pages deploy dist --project-name=emergents-website --branch=main

`wrangler` authenticates as bowen@emergents.io (OAuth). Confirm the release is live by
checking the hashed JS bundle at https://emergents.io matches `dist/assets/index-*.js`.

## Project Architecture

This is a React-based personal website with a glassmorphism design system built using:
- **React 18** with TypeScript
- **Vite** for build tooling
- **Tailwind CSS** with custom design tokens
- **Framer Motion** for animations

### Routing Architecture

The app uses **React Router** with distinct routes:
- `/` - Main homepage (`OnePage.tsx`) with profile and project cards
- `/systemic` - Systemic project landing page (`SystemicLanding.tsx`)
- `/systemic/privacy` - Privacy policy page (`PrivacyPolicy.tsx`)

### Core Navigation Flow
```
App.tsx (BrowserRouter) → Routes
├── "/" → OnePage.tsx (main profile page)
├── "/systemic" → SystemicLanding.tsx
└── "/systemic/privacy" → PrivacyPolicy.tsx
```

### Component Structure

**Layout Components** (`src/components/layout/`):
- `SinglePageLayout`: Full-screen layout with background effects
- `ScrollableLayout`: Scrollable layout for content-heavy pages
- `Container`, `Section`, `Grid`: Basic layout utilities

**UI Components** (`src/components/ui/`):
- `Button`, `Card`, `Badge`: Standard UI elements
- `ProjectCard`, `CompactProjectCard`: Project showcase cards
- `GlowOrb`: Animated background orbs for ambiance
- `Navigation`: Navigation components

**Section Components** (`src/components/sections/`):
- `CompactProfile`: Main profile section
- `EmailCapture`: Waitlist signup form
- `SystemMapDemo`: Interactive demo visualization
- `PersonalHero`, `ProjectsGrid`: Alternative layouts

### Design System

**Custom Tailwind Configuration** (`tailwind.config.js`):
- Dark theme color palette with purple/pink accents
- Glassmorphism utilities (`.glass`, `.glass-heavy`)
- Custom animations (gradient, glow, float, shimmer)
- Typography using Inter font with feature settings

**Global Styles** (`src/styles/globals.css`):
- Glassmorphism component classes
- Gradient text utilities (`.gradient-text`)
- Glow effects and hover states
- Custom scrollbar styling

**Key Design Tokens**:
- Primary colors: `accent-purple`, `accent-pink`, `accent-blue`
- Glass effects: `bg-glass-light`, `backdrop-blur-glass`
- Text hierarchy: `text-primary`, `text-secondary`, `text-muted`

### Project Configuration

- **Styling**: Tailwind CSS with PostCSS
- **Utils**: `cn()` utility using `clsx` and `tailwind-merge` for conditional classes (`src/lib/utils.ts`)
- **Assets**: Public folder for images (`/profile.jpg`, `/emergents-logo.png`)
- **TypeScript**: Strict configuration with Vite compilation

### Key Patterns

1. **React Router Navigation**: Standard routing with `useNavigate` for programmatic navigation
2. **Glassmorphism Design**: Heavy use of backdrop-blur and glass effects
3. **Animation Staging**: Framer Motion with staggered entrance animations
4. **Responsive Design**: Mobile-first approach with `md:` breakpoints
5. **Component Composition**: Reusable layout and UI components with consistent APIs
6. **Project Navigation**: Internal routes use `navigate()`, external links use `window.open()`

---
> Source: [bowen-0/emergents-website](https://github.com/bowen-0/emergents-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
