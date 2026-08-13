## resume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio/resume website built with Next.js 16.1.1, React 19, TypeScript, Tailwind CSS v4, and Framer Motion. The site showcases professional experience, skills, and projects with smooth animations and a dark/light theme toggle.

## Development Commands

```bash
# Start development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run ESLint
npm run lint
```

## Architecture

### Data-Driven Content
All resume content (personal info, skills, experience, education, etc.) is centralized in `src/data/resume.ts`. This is a **single source of truth** for portfolio content - update this file to change displayed information across all sections.

### Component Structure
- **Layout Components** (`src/components/layout/`): Sidebar navigation with theme toggle
- **Section Components** (`src/components/sections/`): Each major section (Hero, About, Skills, Experience, Contact) is a standalone component that imports data from `resume.ts`
- **Animation System** (`src/components/animations/index.tsx`): Reusable Framer Motion animation components:
  - `FadeIn`: Scroll-triggered fade animations with directional support
  - `StaggerContainer` / `StaggerItem`: Sequential animations for lists
  - `Counter`: Animated number counting
  - `HoverScale` / `HoverLift`: Interactive hover effects

### Theme System
The app uses a custom `ThemeProvider` (client component) that:
- Persists theme preference in localStorage
- Defaults to dark mode
- Applies theme class to document root
- Provides `useTheme()` hook for accessing/toggling theme

Theme classes are applied via Tailwind's custom `dark` variant (configured in `globals.css` with `@custom-variant`).

### Styling Approach
- **Tailwind CSS v4** with PostCSS plugin (`@tailwindcss/postcss`)
- Custom CSS variables for background/foreground colors in `globals.css`
- Responsive design with mobile-first approach
- Glassmorphism effects and gradient backgrounds for visual interest

### TypeScript Configuration
- Path alias `@/*` maps to `./src/*` for clean imports
- Strict mode enabled
- JSX transform: `react-jsx` (no React import needed in components)

## File Organization

```
src/
├── app/               # Next.js App Router
│   ├── layout.tsx     # Root layout with metadata and theme provider
│   ├── page.tsx       # Home page (composes all sections)
│   └── globals.css    # Global styles, theme variables, Tailwind imports
├── components/
│   ├── animations/    # Framer Motion animation wrappers
│   ├── layout/        # Sidebar navigation
│   ├── sections/      # Page sections (Hero, About, Skills, etc.)
│   └── ThemeProvider.tsx  # Client-side theme management
└── data/
    └── resume.ts      # All portfolio content (export const objects)
```

## Key Patterns

### Adding New Content
To update portfolio information, modify the exported objects in `src/data/resume.ts`. The section components automatically reflect these changes.

### Creating New Sections
1. Create component in `src/components/sections/`
2. Import data from `@/data/resume`
3. Wrap content with animation components from `@/components/animations`
4. Add to `src/app/page.tsx` in desired order

### Animation Usage
Section components use `FadeIn`, `StaggerContainer`, and other animation components to create scroll-triggered effects. Example pattern:
```tsx
<FadeIn direction="up" delay={0.2}>
  <h2>Section Title</h2>
</FadeIn>
```

### Theme-Aware Styling
Use Tailwind's `dark:` prefix for dark mode variants:
```tsx
className="bg-white dark:bg-neutral-950 text-neutral-900 dark:text-white"
```

## Notes

- This is a single-page application with smooth scrolling navigation
- All external icons are inline SVG components (no icon libraries)
- PDF resume available in `public/resume.pdf`
- Inter font loaded via `next/font/google` in layout

---
> Source: [omarelsaid20/resume](https://github.com/omarelsaid20/resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
