## str8global

> Str8Global is a premium marketing and photography agency website based in Lisbon, Portugal.

# Str8Global - Project Instructions

## Project Overview
Str8Global is a premium marketing and photography agency website based in Lisbon, Portugal.

## Tech Stack
- **Framework:** Next.js 16 (App Router) + React 19 + TypeScript 5
- **Styling:** Tailwind CSS v4 + Styled Components
- **Animations:** GSAP (ScrollTrigger) + Framer Motion + Lenis (smooth scroll)
- **3D:** Three.js + React Three Fiber
- **Package Manager:** npm
- **Icons:** Lucide React

## Language & Communication
- Always respond in **Portuguese (PT-PT)** unless asked otherwise
- All website content is in Portuguese (Portugal)
- Variable names, code comments, and commit messages in English

## Project Structure
```
web-app/src/
├── app/           # Next.js App Router (layout, page, globals.css)
├── components/
│   ├── animations/ # Reusable animated components (GSAP, Framer Motion)
│   ├── effects/    # Visual effects (LightRays, Silk, VolumetricLight)
│   └── sections/   # Page sections (Hero, Services, Portfolio, Contact, etc.)
├── data/           # Mock data and content
├── hooks/          # Custom React hooks
├── lib/            # Utility functions
└── providers/      # Context providers (GSAP, Lenis)
```

## Design System
- **Theme:** Dark (black background, white text)
- **Primary:** #000000 (black)
- **Accent:** #FF10F0 (magenta/hot pink)
- **Typography:** Responsive with clamp()
- **Smooth scroll:** Always use Lenis provider

## Development Commands
```bash
npm run dev    # Start dev server
npm run build  # Production build
npm start      # Start production server
npm run lint   # ESLint check
```

## Coding Conventions
- Use path alias `@/*` for imports from `src/`
- Components use PascalCase filenames
- Hooks use camelCase with `use` prefix
- Prefer Tailwind classes over inline styles
- Support `prefers-reduced-motion` for all animations
- Use `useMediaQuery` hook for responsive logic
- Always lazy-load heavy components (Three.js, videos)

## CARL Integration
- CARL config at `.carl/` — domain-based rules for context-aware responses
- Domains: GLOBAL (always), STACK, ANIMATIONS, DESIGN, CONTENT, RENTAL, PERFORMANCE
- Star-commands: `*brief`, `*deep`, `*fix`, `*new`, `*review`, `*deploy`
- Manage with `*carl` → `*list`, `*add`, `*create`, `*edit`, `*toggle`, `*view`

## Important Notes
- Images are optimized via Next.js Image with allowed domains: `images.unsplash.com`, `player.vimeo.com`
- Video backgrounds have separate mobile/desktop versions
- GSAP ScrollTrigger is initialized via GSAPProvider
- Never commit .env files or credentials

---
> Source: [RenatoWernik/Str8Global](https://github.com/RenatoWernik/Str8Global) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
