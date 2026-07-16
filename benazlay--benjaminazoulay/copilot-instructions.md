## benjaminazoulay

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website for Benjamin Azoulay, a Web3 Full-Stack Engineer. The site showcases projects through an interactive interface with a 3D solar system intro section and draggable project cards.

**Tech Stack**: Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS, Framer Motion

**Domain**: https://benjaminazoulay.com

## Development Commands

```bash
# Development (uses Turbopack)
npm run dev

# Production build
npm run build

# Start production server
npm start

# Linting
npm run lint

# Check for unused dependencies/exports
npm run knip
```

## Pre-commit Hooks

The repository uses Husky with the following automated checks:

1. **lint-staged**: Runs ESLint on staged `.ts` and `.tsx` files
2. **Build validation**: If TypeScript files are staged, runs full production build to catch type errors
3. Commits are blocked if either check fails

## Architecture

### Page Structure

The site is a single-page application with two main sections:

1. **Intro Section** ([page.tsx:19-27](src/app/page.tsx#L19-L27))
   - 3D solar system animation with interactive planets (GitHub, LinkedIn, Hashnode)
   - Mouse-tracking 3D rotation effect
   - Scroll widget to navigate to projects

2. **Projects Section** ([page.tsx:29-45](src/app/page.tsx#L29-L45))
   - Reorderable project cards using Framer Motion
   - Each project has its own component with custom content
   - Drag-and-drop enabled on desktop only

### Project System

Projects follow a data-driven architecture:

- **[projectData.ts](src/app/projects/projectData.ts)**: Central registry defining all projects with title, logo, color, and content component
- **Individual Project Components**: Each project (e.g., `StargateProject.tsx`, `RiskophobeProject.tsx`) exports a component that receives a `color` prop
- **[ProjectList.tsx](src/app/projects/ProjectList.tsx)**: Manages state and reordering logic using Framer Motion's `Reorder` components
- **[ProjectBox.tsx](src/app/components/ProjectBox.tsx)**: Reusable wrapper that renders project logo, border styling, and content

**To add a new project:**
1. Create a new component in `src/app/projects/` (e.g., `NewProject.tsx`) that accepts `ProjectContentProps`
2. Import and add to the `projectData` array in [projectData.ts](src/app/projects/projectData.ts)
3. Add the project logo to `/public/logos/`

### Key Components

- **[SolarSystem.tsx](src/app/components/SolarSystem.tsx)**: 3D animated intro with orbital planets and mouse-tracking rotation
- **[ProjectBox.tsx](src/app/components/ProjectBox.tsx)**: Card wrapper with dynamic border colors and semi-circular cutout design
- **[ScrollWidget.tsx](src/app/components/ScrollWidget.tsx)**: Client-side navigation button (dynamically imported)
- **[useIsMobile.ts](src/app/useIsMobile.ts)**: Hook for responsive behavior (disables drag on mobile)

### Styling

- Uses Tailwind CSS with custom configuration
- Theme color: `#7f1d1d` (red-900)
- Custom gradient backgrounds (`from-red-900 to-gray-800`)
- Dynamic border colors based on project data
- CSS animations for orbital motion defined inline

### Font Configuration

The site uses Google Fonts via `next/font`:
- **Geist Sans**: Primary font (`--font-geist-sans`)
- **Geist Mono**: Monospace font (`--font-geist-mono`)

Fonts are loaded in [layout.tsx](src/app/layout.tsx) and applied via CSS variables.

### SEO & Metadata

Comprehensive metadata configuration in [layout.tsx](src/app/layout.tsx:15-57) includes:
- OpenGraph tags for social sharing
- Twitter card configuration
- Structured data with keywords focused on Web3/blockchain development
- Favicon and PWA manifest support

### Production Optimizations

The [next.config.ts](next.config.ts) includes:
- Compression enabled
- SWC minification
- Console statements removed in production builds

## Path Aliases

- `@/*` maps to `./src/*` (configured in [tsconfig.json](tsconfig.json))

## Unused Code Detection

Knip is configured to detect unused dependencies and exports. Configuration in [knip.json](knip.json) ignores build tools like `autoprefixer` and `postcss-load-config` which are used implicitly.

## Notes

- All interactive components are client-side (`"use client"` directive)
- Images should be placed in `/public/` and referenced with root-relative paths
- The drag-and-drop functionality is disabled on mobile for better UX

---
> Source: [BenAzlay/benjaminazoulay](https://github.com/BenAzlay/benjaminazoulay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
