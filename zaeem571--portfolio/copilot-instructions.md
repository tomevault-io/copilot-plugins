## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A React portfolio website for Zaeem Mahmood showcasing projects, work experience, and skills. The site is being migrated from component-level CSS to Tailwind CSS and is deployed to GitHub Pages at https://zaeem571.github.io/portfolio/.

**Tech Stack:** React 18.2.0 | Tailwind CSS 3.4.19 | craco (Create React App config override) | react-icons | react-type-animation

## Skills & Technologies

**Frontend:**
- React 18+ (functional components, hooks, React.memo for optimization)
- Tailwind CSS 3.4+ (utility-first styling, responsive design, custom theme configuration)
- CSS-in-JS concepts (component-scoped styling via Tailwind utilities)
- JavaScript ES6+ (arrow functions, destructuring, spread operators)

**State & Lifecycle:**
- React hooks (useState, useEffect, useCallback, useContext)
- Custom hooks (useIntersectionObserver for scroll animations and scroll spy)
- Component composition and reusability patterns

**Performance:**
- React.memo() for preventing unnecessary re-renders
- IntersectionObserver API for efficient scroll-triggered animations
- RequestAnimationFrame for smooth scroll performance
- Code splitting and lazy loading via create-react-app

**Build & Tooling:**
- Create React App with craco configuration override
- PostCSS for Tailwind CSS compilation
- npm package management
- GitHub Pages deployment (gh-pages)
- Git version control

**UI/UX:**
- Responsive mobile-first design (xs, md, lg breakpoints)
- Dark theme implementation
- Smooth animations and transitions (fade-in, scroll animations)
- Accessible navigation with scroll spy functionality
- Mobile menu toggle and navigation

**Libraries & Dependencies:**
- **react-icons**: Icon sets for social links and UI elements
- **react-type-animation**: Animated text typing effect
- **gh-pages**: GitHub Pages deployment automation

## Architecture

The app follows a simple component hierarchy:

- **App.js**: Main entry point that composes all page sections in order
- **src/components/**: Functional components for each section:
  - `Navbar.js`: Navigation with scroll spy and mobile menu toggle
  - `About.js`: Hero/intro section
  - `WorkExperience.js`: Work history using Card component
  - `Projects.js`: Project showcase using Card component
  - `CertificateAndInvolvement.js`: Certificates and involvement
  - `Contact.js`: Contact information and call-to-action
  - `Card.js`: Reusable card component for projects and work experience
  - `useIntersectionObserver.js`: Custom hook for scroll animations
  - `ErrorBoundary.js`: Error handling wrapper

**Styling Architecture:**

- **Tailwind CSS**: Primary styling method (fully migrated from component CSS)
- **src/config/colors.js**: Centralized color constants—**source of truth for all colors**. Modify this single file to update the entire app's color scheme:
  - Background colors (primary, app, section, card, cardHover)
  - Navigation colors (default, thin, mobile)
  - Text colors (primary, muted, heading)
  - Brand/Primary colors (default, hover, light, lighter)
- **tailwind.config.js**: Imports and applies colors from `src/config/colors.js`:
  - Spacing: `sm` (9px), `md` (18px), `lg` (27px)
  - Font: Quicksand family loaded from Google Fonts
  - Animation: `fade-in` keyframe for entrance animations
  - Responsive breakpoints: `xs` (480px), `md` (768px), `md2` (960px)
- **App.css**: Contains only Tailwind directives (@tailwind rules) and global focus styles
- **craco.config.js**: Minimal config—PostCSS handles Tailwind compilation via postcss.config.js

**Key Patterns:**

- Components use React.memo() for performance (Navbar, Card)
- Navbar uses IntersectionObserver for scroll spy and requestAnimationFrame for scroll performance
- Mobile-first responsive design with Tailwind breakpoints
- Responsive text sizes, padding, and layout (see Card.js for extensive examples)

## Common Commands

**Development:**
```bash
npm run dev              # Start dev server at http://localhost:3000
npm run build           # Build for production
npm run test            # Run Jest tests
npm run deploy          # Deploy to GitHub Pages (predeploy runs build)
```

**Package Manager:** The repo uses both npm and pnpm (lockfiles present for both). Use `npm` for consistency with package.json scripts.

## Key Files & When to Modify

- **src/config/colors.js**: ⭐ **Change colors here only**—all colors derive from this single source. Modify color values to update the entire site's theme instantly.
- **tailwind.config.js**: Add new theme extensions (spacing, animations, breakpoints); colors are auto-imported from src/config/colors.js
- **src/App.js**: Add/remove/reorder page sections
- **src/components/Card.js**: Primary reusable component for content cards; heavily tailored with responsive classes
- **src/components/Navbar.js**: Navigation, mobile menu, scroll spy logic
- **public/index.html**: Global fonts (Google Fonts), meta tags, page title, favicon

## Styling System

**Centralized Color Management:**
All colors are defined in `src/config/colors.js`. To change the color scheme:
1. Edit color values in `src/config/colors.js`
2. Colors automatically cascade to `tailwind.config.js`
3. All components instantly use the updated colors (no component changes needed)

**When Adding Features or Fixing Styles:**
- Use Tailwind utility classes directly in className attributes
- For colors, use the custom classes defined in tailwind.config.js (e.g., `bg-card`, `text-heading`, `bg-primary`)
- For color values, always reference `src/config/colors.js`—never hardcode hex values or rgba() in components
- Use responsive prefixes (`xs:`, `md:`, `md2:`) for mobile/tablet/desktop breakpoints
- Avoid creating new CSS files unless required for global animations or shared utility styles

## Deployment

The site is deployed via GitHub Pages with gh-pages package:

```bash
npm run deploy          # Builds and deploys to gh-pages branch
```

The homepage is set to `http://zaeem571.github.io/portfolio` in package.json. Navigation links use hash routing (e.g., `#about`, `#projects`) to scroll to sections.

## Common Issues & Solutions

**Build fails after editing Tailwind classes:** Clear build cache with `rm -rf build` and rebuild.

**Responsive layout broken:** Check that responsive prefixes match actual Tailwind breakpoints (default: `xs` < 640px, `md` ≥ 768px). Test across screen sizes in browser DevTools.

**Scroll spy not highlighting correct section:** Verify that section IDs in HTML match NAV_ITEMS IDs in Navbar.js, and that IntersectionObserver rootMargin values are appropriate for viewport height.

---
> Source: [zaeem571/portfolio](https://github.com/zaeem571/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
