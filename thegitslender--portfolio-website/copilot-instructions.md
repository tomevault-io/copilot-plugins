## portfolio-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (http://localhost:5173)
npm run build    # Build for production
npm run preview  # Preview production build
npm run lint     # Run ESLint
```

## Architecture

This is a React portfolio website for Hany El Atlassi (AI Engineer) using Vite, Tailwind CSS v4, and Framer Motion.

### Routing Structure
- `/` - Home page (composes all portfolio sections)
- `/project/:id` - Individual project detail pages
- `*` - 404 Not Found

### Key Architectural Patterns

**Layout System**: `App.jsx` wraps all routes in `Layout.jsx`, which provides the sticky Header and Footer. BrowserRouter is in `main.jsx`.

**Page Transitions**: Uses Framer Motion's `AnimatePresence` with `mode="wait"` for smooth page transitions.

**Data Separation**: All content lives in `/src/data/` files (personal.js, experience.js, projects.js, skills.js, certifications.js, contact.js). Components import and render this data - never hardcode content in components.

**Tailwind v4 Configuration**: Uses CSS-based config with `@theme` directive in `index.css`. Custom design tokens (colors, shadows, typography) are defined there using CSS custom properties like `--color-accent-blue`.

### Component Organization

- `/components/layout/` - Header, Footer, Layout wrapper
- `/components/sections/` - Hero, About, Experience, Projects, Skills, Certifications, Contact
- `/components/ui/` - Reusable primitives (Button, Card, Badge, SectionHeading, ProjectCard, AnimatedSection)
- `/components/effects/` - Animation wrappers (ParallaxSection, MouseFollower)
- `/pages/` - Route-level components (Home, ProjectDetail, NotFound)
- `/hooks/` - Custom hooks for scroll animations and parallax
- `/utils/` - Framer Motion animation variants and helpers

### Design System

Apple-inspired aesthetic with tech accents. Key CSS custom properties:
- Colors: `--color-primary-text`, `--color-accent-blue`, `--color-surface-card`
- Shadows: Layered Formix-style (`--shadow-md`, `--shadow-lg`)
- Use `container-main` class for consistent max-width container

### Animation Guidelines

Use Framer Motion for all animations. Key patterns:
- Scroll-triggered: `useInView` hook with threshold 0.2-0.3
- Page transitions: variants with opacity/y transforms
- Hover effects: `whileHover` with scale and shadow changes
- Stagger children: `staggerChildren: 0.1` in container variants

Components should check `prefers-reduced-motion` and disable animations accordingly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheGitSlender) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
