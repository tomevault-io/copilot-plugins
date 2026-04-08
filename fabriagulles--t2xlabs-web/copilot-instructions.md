## t2xlabs-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

t2xlabs is a marketing/landing page website for an AI automation company. Live at https://www.t2xlabs.com/

## Commands

```bash
npm run dev       # Start dev server on port 8080
npm run build     # Production build
npm run lint      # Run ESLint
npm run preview   # Preview production build
```

## Tech Stack

- **Build:** Vite 5.4 with SWC
- **Framework:** React 18 + TypeScript
- **Styling:** Tailwind CSS with custom futuristic theme (neon-cyan, cyber-blue, matrix-green)
- **UI Components:** shadcn/ui (Radix-based, located in `src/components/ui/`)
- **Forms:** React Hook Form + Zod validation
- **State:** React Query for server state, localStorage for preferences

## Architecture

### Path Alias
`@/*` resolves to `./src/*`

### Project Structure
- `src/pages/` - Route pages (Index.tsx is the main landing page)
- `src/components/` - Page sections (HeroSection, ServicesIA, ContactForm, etc.)
- `src/components/ui/` - shadcn/ui primitives (50+ components)
- `src/hooks/` - Custom hooks (use-toast, use-mobile)
- `src/lib/utils.ts` - `cn()` utility for Tailwind class merging

### Page Composition
The landing page (`Index.tsx`) composes section components. Each section uses HTML anchor IDs (hero, advantages, services, clients, contact) for smooth scroll navigation.

### External Integrations
- **Airtable:** ContactForm submits to Airtable API
  - Requires `VITE_AIRTABLE_BASE_ID` and `VITE_AIRTABLE_TOKEN` env vars
- **Google Tag Manager:** GTM-N56QX78D for analytics

### Cookie Consent System
GDPR-compliant cookie banner with preferences stored in localStorage. Privacy modal uses React Portal for proper z-index handling.

## Styling Conventions

The design uses a futuristic dark theme with custom CSS variables (HSL-based). Key custom classes defined in `tailwind.config.ts`:
- Colors: `space-black`, `cyber-blue`, `neon-cyan`, `matrix-green`, `plasma-purple`
- Animations: `glow-pulse`, `float`, `matrix`, `typewriter`
- Fonts: Inter (display), JetBrains Mono (code)

## Language

Site content is in Spanish. The HTML lang attribute is set to "es".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FabriAgulles)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FabriAgulles)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
