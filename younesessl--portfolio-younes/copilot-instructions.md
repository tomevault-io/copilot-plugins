## portfolio-younes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 16 portfolio application built with React 19, TypeScript, and Tailwind CSS v4. The project uses the App Router architecture and is configured for modern web development with animation libraries (AOS and GSAP) ready for integration.

## Development Commands

### Running the Application
```bash
npm run dev    # Start development server at http://localhost:3000
npm run build  # Create production build
npm start      # Run production server
npm run lint   # Run ESLint
```

### Package Management
```bash
npm install              # Install dependencies
npm install <package>    # Add new package
```

## Architecture

### Next.js App Router Structure

- **App Router** (Next.js 16): Uses `src/app/` directory for file-based routing
- **Server Components by default**: All components are Server Components unless marked with `'use client'`
- **Root Layout** (`src/app/layout.tsx`): Defines HTML structure, loads fonts (Geist Sans and Geist Mono), and wraps all pages
- **Pages**: Each `page.tsx` file in `src/app/` becomes a route

### Component Strategy

**Current state**: Minimal component structure (just layout and homepage)

**When adding components**:
- Create `src/components/` directory for reusable UI components
- Use `'use client'` directive for interactive components requiring hooks or browser APIs
- Keep Server Components as the default for better performance
- Use TypeScript interfaces for component props

**Path Aliases**: Import using `@/*` prefix (e.g., `import Component from '@/components/Component'`)

### Animation Libraries

Two animation libraries are installed but not yet integrated:

- **AOS (Animate On Scroll)** v2.3.4: For scroll-triggered animations
  - Requires `'use client'` directive
  - Must initialize with `AOS.init()` in a client component

- **GSAP** v3.13.0: For complex timeline-based animations
  - Requires `'use client'` directive
  - Use `useGSAP()` hook from `@gsap/react` for proper React integration (install if needed)

**Integration approach**: Create a client-side wrapper component in layout or use client components for animated sections.

## Styling with Tailwind CSS v4

### Configuration
- Uses **Tailwind CSS v4** with PostCSS plugin (`@tailwindcss/postcss`)
- No separate `tailwind.config.js` needed—configuration is in `globals.css` using `@theme inline`
- Import Tailwind with `@import "tailwindcss"` in CSS files

### Theme System
- **CSS Variables** defined in `src/app/globals.css`:
  - `--background` and `--foreground` for theming
  - Font variables: `--font-geist-sans` and `--font-geist-mono`
- **Dark Mode**: Automatic support via `prefers-color-scheme` media query
  - Use `dark:` prefix for dark mode utilities (e.g., `dark:bg-black`)

### Styling Conventions
- Use Tailwind utility classes in JSX (e.g., `className="flex items-center gap-4"`)
- Responsive design with breakpoints: `sm:`, `md:`, `lg:`, etc.
- Font variables applied in layout.tsx: Use `font-sans` or `font-mono` classes

## TypeScript Configuration

- **Strict mode enabled**: All code must satisfy strict TypeScript checks
- **Target**: ES2017
- **JSX**: react-jsx (modern JSX transform)
- **Module resolution**: bundler (Next.js optimized)
- **Path alias**: `@/*` maps to `./src/*`

### Type Definitions
- Use `type` for object shapes and unions
- Use `interface` for component props (extendable)
- Leverage Next.js types: `Metadata`, `NextConfig`, etc.

## Code Organization Patterns

### Current Structure
```
src/
├── app/
│   ├── layout.tsx      # Root layout with fonts and metadata
│   ├── page.tsx        # Homepage
│   └── globals.css     # Global styles and Tailwind config
```

### Recommended Future Structure
```
src/
├── app/              # Routes and layouts
├── components/       # Reusable UI components
├── hooks/           # Custom React hooks
├── lib/             # Third-party library configurations
├── utils/           # Helper functions
└── types/           # Shared TypeScript types
```

## Working with Next.js 16 Features

### Image Optimization
- Use `next/image` component for all images (see page.tsx:1)
- Set `priority` for above-the-fold images
- Images in `/public` are served from root URL (e.g., `/next.svg`)

### Fonts
- Google Fonts loaded via `next/font/google` in layout.tsx
- Fonts create CSS variables passed to components via `variable` property
- Apply font variables using `className` prop

### Metadata
- Export `metadata` object from layout.tsx or page.tsx for SEO
- Use `Metadata` type from "next" for type safety

### Server vs Client Components
- **Server Components** (default):
  - Faster performance, smaller bundle
  - Can fetch data directly, no `useEffect` needed
  - No access to browser APIs or React hooks

- **Client Components** (`'use client'`):
  - Use for interactivity, hooks, browser APIs
  - Required for animations (AOS, GSAP)
  - Place directive at top of file

## Development Workflow

### Adding New Features
1. Determine if component needs client-side interactivity
2. Create component in appropriate directory (`src/app/` for pages, `src/components/` for reusable components)
3. Use path aliases (`@/`) for imports
4. Apply Tailwind utilities for styling
5. Ensure TypeScript types are defined
6. Test in both light and dark modes

### Working with Animations
1. Create client component with `'use client'`
2. For AOS: Initialize in `useEffect` hook
3. For GSAP: Use `useGSAP()` hook or `useEffect` for animations
4. Ensure animations respect user's motion preferences (prefers-reduced-motion)

### ESLint
- Uses ESLint 9 flat config format (`eslint.config.mjs`)
- Includes Next.js Core Web Vitals rules
- TypeScript rules enabled
- Ignores: `.next/`, `out/`, `build/`, `next-env.d.ts`

## Best Practices

- **Server-first**: Use Server Components by default, add `'use client'` only when necessary
- **Import optimization**: Use path aliases (`@/`) consistently
- **Type safety**: Define explicit types for all component props and function parameters
- **Responsive design**: Mobile-first approach with Tailwind breakpoints
- **Accessibility**: Use semantic HTML and ARIA attributes where appropriate
- **Dark mode**: Test all UI changes in both light and dark modes
- **Font variables**: Use `font-sans` and `font-mono` classes instead of arbitrary font families

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YounesEssl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YounesEssl)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
