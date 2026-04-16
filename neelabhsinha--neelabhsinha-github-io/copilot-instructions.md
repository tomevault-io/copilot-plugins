## neelabhsinha-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website built with React 19, TypeScript, Vite, and Material-UI. Features a single-page application with smooth Framer Motion animations, dark/light theme support, and modular data-driven content management.

## Development Commands

```bash
# Install dependencies
npm install

# Setup MCP configuration (first time only)
cp .mcp.example.json .mcp.json
# Then edit .mcp.json and add your Context7 API key

# Development server (runs on http://localhost:5173)
npm run dev

# Production build (outputs to dist/)
npm run build

# Preview production build locally
npm run preview

# Run linter
npm run lint
```

## MCP Configuration

This project uses MCP (Model Context Protocol) for enhanced Claude Code integration:

1. **Setup**: Copy `.mcp.example.json` to `.mcp.json`
2. **API Key**: Get your Context7 API key from https://context7.com
3. **Configure**: Replace `your-api-key-here` with your actual key in `.mcp.json`
4. **Security**: `.mcp.json` is gitignored to protect your API key

## Architecture Overview

### Data-Driven Content System

All portfolio content is managed through separate TypeScript files in `src/data/`:
- Each data module exports typed data (e.g., `publications.ts`, `projects.ts`, `experience.ts`)
- `src/data/profile.ts` aggregates all data modules into a single `ProfileData` object
- All types are defined in `src/types/index.ts`
- This architecture allows content updates without touching component code

### Theme System

The theme system is centralized with a custom hook:
- `src/data/theme.ts` exports `lightTheme` and `darkTheme` configurations
- `src/hooks/useTheme.tsx` provides the `ThemeProvider` and `useTheme` hook
- Theme preference persists in localStorage
- MUI theme is dynamically generated from theme config
- Components access theme via `useTheme()` hook for `isDark` boolean and `toggleTheme()` function

### Component Architecture

- **Layout Components** (`src/components/Layout/`): Header with navigation and theme toggle
- **Section Components** (`src/components/Sections/`): Self-contained page sections (Hero, About, Education, Experience, Publications, Projects, Skills, Certifications, Contact)
- All sections receive data as props from `profileData` in `App.tsx`
- Each section is responsible for its own layout and styling using MUI's `sx` prop

### Animation System

Framer Motion animations are centralized in `src/utils/animations.ts`:
- Exports reusable animation variants: `fadeInUp`, `fadeInDown`, `fadeInLeft`, `fadeInRight`, `scaleIn`, `staggerContainer`, `slideInFromBottom`
- `createDelayedAnimation()` helper for timed animations
- Components use `react-intersection-observer` for scroll-triggered animations

### App Structure

`src/App.tsx` is the main entry point:
1. Wraps app in `ThemeProvider`
2. `AppContent` component uses `useTheme()` to access theme state
3. Renders `Header` and all section components in order
4. Passes data from `profileData` to each section

## Content Management

### Adding New Content

All content additions are made by editing TypeScript files in `src/data/`:

**Projects** (`src/data/projects.ts`): Add to `projects` array with required fields: `id`, `title`, `summary`, `date`, `tags`, `links`, `category`. Optional: `description`, `image`, `featured`.

**Publications** (`src/data/publications.ts`): Add to `publications` array with required fields: `id`, `title`, `authors`, `date`, `venue`, `abstract`, `tags`, `featured`, `links`. Optional: `venueShort`, `image`.

**Experience** (`src/data/experience.ts`): Add to `experience` array with required fields: `id`, `position`, `company`, `location`, `startDate`, `endDate`, `description`. Optional: `companyUrl`, `companyLogo`, `current`.

**Education** (`src/data/education.ts`): Add to `education` array with required fields: `id`, `degree`, `institution`, `location`, `startDate`, `endDate`. Optional: `gpa`, `courses`, `description`, `url`, `institutionLogo`.

**Skills** (`src/data/skills.ts`): Add to `skills` array with `id`, `name`, and `items` (array of skills with `name` and `icon`).

**Certifications** (`src/data/certifications.ts`): Add to `certifications` array with required fields: `id`, `title`, `issuer`, `date`. Optional: `url`, `icon`.

**Personal Info** (`src/data/personal.ts`): Single object with `name`, `title`, `bio`, `avatar`, `interests`, `currentFocus`, `hobbies`, `social`.

### Creating New Sections

1. Create component in `src/components/Sections/YourSection.tsx`
2. Add data structure to `src/types/index.ts` if needed
3. Create data file in `src/data/yourData.ts`
4. Import and add data to `profileData` in `src/data/profile.ts`
5. Import section component in `src/App.tsx` and render with data prop

## Build & Deployment

### Build Configuration

- Vite config in `vite.config.ts` sets `base: '/'` and `outDir: 'dist'`
- TypeScript compilation runs before Vite build: `tsc -b && vite build`
- Build outputs to `dist/` directory

### GitHub Actions Deployment

Deployment workflow (`.github/workflows/deploy.yml`) triggers on:
- Push to `main` branch
- Manual workflow dispatch

Build process:
1. Node.js 22.12.0 setup with npm cache
2. `npm ci` for clean install
3. `npm run lint` (if present)
4. `npm run build`
5. Upload `dist/` as artifact
6. Deploy to GitHub Pages

**Important**: Only push to `main` when ready to deploy, as it triggers automatic deployment.

## TypeScript Configuration

- `tsconfig.json`: Base config
- `tsconfig.app.json`: App-specific config with strict mode
- `tsconfig.node.json`: Node/Vite config
- All types in `src/types/index.ts` for centralized type management

## Styling Approach

- MUI's `sx` prop for component-level styling
- No separate CSS modules or styled-components
- Theme values accessed via MUI theme object
- Responsive breakpoints: `theme.breakpoints.down('md')`, etc.
- Background gradients defined inline in `App.tsx` based on `isDark` state
- Always use descriptive variable names

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neelabhsinha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
