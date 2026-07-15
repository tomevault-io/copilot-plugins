## myporto

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a modern portfolio website for Tommy Nurwantoro, an Engineering Manager. It's a single-page application (SPA) built with React 19, TypeScript, Vite, Framer Motion, and Tailwind CSS v4, featuring a dark theme with emerald accents, smooth animations, and interactive components.

**Key Architecture:**
- **Component organization:** Reusable UI components in `src/components/ui/` and page-specific sections in `src/components/sections/`
- **Centralized data:** All content data (projects, experiences, education, skills) is in `src/constants/data.ts`
- **Type safety:** Comprehensive TypeScript interfaces in `src/types/index.ts`
- **Custom hooks:** `useInView` for scroll-triggered animations using Intersection Observer

## Development Commands

```bash
npm run dev      # Start development server (runs on http://localhost:5173)
npm run build    # TypeScript compile + Vite build for production
npm run preview  # Preview production build locally
npm run lint     # Run ESLint
```

**Docker:**
```bash
docker build -t myporto .
docker run -p 3001:3001 myporto  # Container listens on port 3001 internally
```

The production Docker container serves the built static files via Nginx on port 3001 (as configured in the Dockerfile).

## Content Management

When updating content, edit these files:

- **`src/constants/data.ts`** - All content data (projects, experiences, education, skills)
- **`src/constants/theme.ts`** - Theme constants for colors, spacing, typography
- **`tailwind.config.js`** - Custom animations and Tailwind extensions

**Important:** Project images are imported in `src/constants/data.ts` and stored in `src/assets/`. When adding new projects, import the image at the top of the file and include it in the project object.

## Component Architecture

### UI Components (`src/components/ui/`)

These are reusable, design-system components:
- **Modal** - Accessible modal with focus management
- **Button** - Variants: primary, secondary, ghost, icon
- **Card** - Container component with consistent styling
- **ProgressBar** - Animated progress bars for skills
- **Section** - Wrapper for page sections with in-view animation
- **AnimatedBackground** - Mouse-tracking gradient background
- **ParticleBackground** - Floating particle effects

### Page Sections (`src/components/sections/`)

These compose the main page:
- **Header** - Hero section with introduction
- **Skills** - Skill categories with progress bars
- **Projects** - Project grid with modal detail view
- **Experience** - Work history timeline
- **Education** - Educational background
- **Contact** - Contact information and links

### Utility Components

- **Navigation** - Fixed nav bar with smooth scroll
- **SchedulePage** - Calendly integration page
- **PaymentPage** - Payment/support page

## Animation System

The app uses multiple animation layers:

1. **Framer Motion animations:** Primary animation system for sophisticated effects
   - Typewriter effects on headings
   - 3D tilt effects on hover
   - Staggered entrance animations with variants
   - Scroll-synchronized timeline animations
2. **Scroll-triggered animations:** The `useInView` hook (src/hooks/useInView.ts) uses Intersection Observer to add `data-in-view` attributes when elements enter viewport (used in Projects component)
3. **Mouse-tracking:** `AnimatedBackground` follows mouse position (managed in App.tsx state)
4. **Custom animations:** Defined in `tailwind.config.js` (gradient animations, floats, pulses)
5. **CSS animations:** Global styles in `src/index.css` (gradient-text animation)

## Framer Motion Integration

The app uses Framer Motion for sophisticated animations:

1. **Custom hooks:** Located in `src/hooks/framer/`
   - `useTypewriter` - Cinematic text typing effects with cursor
     - Parameters: `text`, `speed` (default 50ms), `delay`, `loop`
     - Returns: `displayedText`, `isTyping`, `isComplete`, `reset`
     - Example: Used in Header for typing "Engineering Manager" title
   - `useTilt` - 3D perspective tilt on mouse movement
     - Parameters: `tiltRange` (default 8deg), `smoothing` (default 0.1)
     - Returns: `rotateX`, `rotateY`, `scale` MotionValues, `ref`, `reset`
     - Use with `useMotionTemplate` for transform string
     - Example: Used on profile picture for interactive 3D effect
   - `useTimeline` - Scroll-synchronized timeline animations
     - Parameters: `containerRef`, `itemCount`, `offset`
     - Returns: `progress` (0 to itemCount), `scrollYProgress`, `containerRef`
     - Maps scroll position to item progress for multi-step animations

2. **Component patterns:**
   - Use `motion.div` to wrap elements for animation
   - `AnimatePresence` for enter/exit transitions
   - `variants` for stagger children animations
   - `whileInView` for scroll-triggered animations
   - `useMotionTemplate` for dynamic transform strings combining MotionValues

3. **Performance:**
   - `viewport={{ once: true }}` prevents replay
   - `layout` prop for smooth layout animations
   - `useMotionValue` for tracked values (mouse, scroll)
   - `requestAnimationFrame` in useTilt for smooth 60fps updates

**When adding animations:**
- Prefer `motion.*` components over CSS for complex sequences
- Keep simple hover effects as CSS (better performance)
- Always set `viewport={{ once: true }}` for scroll animations
- Use `staggerChildren` for cascading effects
- Use MotionValues with `useTransform` for derived values

## Routing

Uses React Router DOM v7 for client-side routing:
- Main page is at `/`
- Schedule page at `/schedule`
- Payment page at `/payment`

## Styling Conventions

- **Tailwind v4** utility-first approach
- **Color palette:** Dark theme (gray-950, gray-900) with emerald accents (emerald-500, emerald-400)
- **Responsive:** Mobile-first with `md:` and `lg:` breakpoints
- **Accessibility:** Semantic HTML, ARIA labels, keyboard navigation, skip-to-content link
- **className merging:** Use the `cn()` utility from `src/utils/cn.ts` (combines clsx + tailwind-merge)

## Tech Stack Details

- **React 19** - Latest React with concurrent features
- **Vite 7** - Build tool and dev server (not Webpack)
- **TypeScript 5.9** - Strict type checking with separate configs for app and Node.js
- **Framer Motion** - Production-ready motion library for React (animations, gestures, transitions)
- **Tailwind CSS 4** - Latest version with PostCSS integration
- **Lucide React** - Icon library (icons are imported as components)

## Build Output

- Development: Vite dev server with HMR
- Production: Static files in `dist/` directory
- Docker: Multi-stage build with Node.js builder → Nginx production server

## Key Patterns

1. **State management:** Local React state (no Redux/Zustand)
2. **Animation state:** Framer Motion hooks (useTypewriter, useTilt, useTimeline) for complex animations
3. **Data flow:** Props drilling for component communication
4. **Icons:** Import from `lucide-react` as components
5. **Type imports:** Use `import type { }` for type-only imports
6. **Ref forwarding:** Components use `React.forwardRef` when refs are needed
7. **Motion values:** Use `useMotionValue` and `useTransform` for derived animation values

## Future Development

The following directories exist for planned features:
- **`src/components/modern/`** - Reserved for modernized/refactored components
- **`src/hooks/gsap/`** - Reserved for GSAP (GreenSock) animation hooks (currently the app uses Framer Motion for animations)

---
> Source: [tommynurwantoro/myporto](https://github.com/tommynurwantoro/myporto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
