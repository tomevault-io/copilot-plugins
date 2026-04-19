## solutionsimpactweb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a bilingual (French/English) digital marketing agency website for Quebec businesses, built with Next.js 15 and TypeScript. The project focuses on landing page creation, SEO services, and AI-powered marketing solutions.

### Business Carousel Feature
The homepage features an interactive carousel showcasing 10 distinct website designs for different business types:
- AI Marketing Agency (original design)
- Medical Clinic with appointment booking
- Antique Shop with catalog browsing  
- Auto Garage with service booking
- Retail Store with product showcase
- Restaurant with menu and reservations
- Law Firm with consultation forms
- Beauty Salon with service packages
- Pet Grooming with care packages
- Home Services with maintenance offerings

## Architecture

### Core Structure
- **Framework**: Next.js 15 with App Router
- **Languages**: TypeScript with strict configuration (noUnusedLocals, noUnusedParameters, exactOptionalPropertyTypes)
- **Styling**: Tailwind CSS v4 with PostCSS
- **Internationalization**: next-intl for i18n routing and content
- **Animations**: Framer Motion for UI animations
- **Error Tracking**: Sentry integration for production monitoring

### Key Directories
- `web/` - Main Next.js application (all npm commands run from here)
- `web/app/[locale]/` - Internationalized route structure
- `web/messages/` - Translation files (fr.json, en.json)
- `web/components/` - React components including BusinessCarousel, ErrorBoundary, compliance tools
- `web/content/` - MDX content files for guides and documentation
- `web/lib/` - Utility functions and configurations
- `Documents/` - Business documentation (BusinessPlanV0.5.md, roadmap)
- `Plans/` - Project improvement plans

### Internationalization Setup
- **Default locale**: French (`fr`)
- **Supported locales**: `['fr', 'en']`
- **Routing**: Always prefixed (`/fr/`, `/en/`)
- **Middleware**: Automatic locale detection and routing
- **Configuration**: `i18n.ts` and `i18n/request.ts` handle locale resolution

## Development Commands

All commands must be run from the `web/` directory:

```bash
cd web

# Development
npm run dev                    # Start dev server with Turbopack
npm run build                  # Production build
npm run start                  # Start production server

# Testing
npm run test                   # Run Vitest tests
npm run test:ui                # Vitest UI
npm run test:run               # Run tests once
npm run test:a11y              # Run Playwright accessibility tests
npm run test:a11y:headed       # Run accessibility tests with browser

# Linting & Validation
npm run lint                   # Run ESLint
npm run lint:fix               # Auto-fix ESLint issues
npm run i18n:lint              # Validate translation files
npm run content:validate       # Validate content structure

# Analysis
npm run analyze                # Bundle analyzer for both client and server
npm run analyze:server         # Server bundle analysis
npm run analyze:browser        # Browser bundle analysis
```

## Testing Strategy

- **Unit Tests**: Vitest with React Testing Library (`__tests__/` directory)
- **Accessibility Tests**: Playwright with axe-core (`tests/accessibility.spec.ts`)
- **Test Config**: `vitest.config.ts` with jsdom environment
- **Run single test**: `npm run test -- path/to/test.spec.ts`

## Code Quality

### ESLint Configuration
- Extends Next.js core-web-vitals and TypeScript configs
- Import sorting with `simple-import-sort` plugin
- Consistent type imports enforced
- Custom rules in `eslint.config.mjs`

### Git Hooks
- **Husky**: Pre-commit hooks configured
- **Lint-staged**: Runs ESLint on staged files before commit
- Configuration in `package.json` lint-staged section

### TypeScript Configuration
- Strict mode enabled with additional checks
- Path alias: `@/*` maps to project root
- Incremental compilation for faster builds
- Excludes test files and scripts from compilation

## Content Management

### MDX Support
- MDX files supported for content pages
- Configuration in `next.config.ts` with `@next/mdx`
- Content directory structure in `web/content/`
- Frontmatter parsing with `gray-matter`

### Structured Content
- Compliance guides in `content/compliance/`
- Pricing information in `content/pricing/`
- Service definitions in `content/services.fr.json`

## Key Features

### Interactive Components
- **BusinessCarousel**: Main homepage carousel with 10 business designs
- **AIReadinessAssessment**: Interactive assessment tool
- **ComplianceHeatmap**: Visual compliance tracking
- **PricingCalculator**: Dynamic pricing tool
- **LeadCaptureForm**: Lead generation with validation

### Performance Optimizations
- **LazyMotion**: Lazy-loaded Framer Motion features
- **OptimizedImage**: Image optimization wrapper
- **ProgressiveEnhancement**: Graceful degradation support
- **Bundle Analysis**: Configured for monitoring bundle sizes

### SEO & Analytics
- **StructuredData**: JSON-LD structured data generation
- **PageViewTracker**: Analytics integration
- **Canonical & HrefLang**: SEO link components
- **Sitemap & Robots**: Auto-generated from `app/sitemap.ts` and `app/robots.ts`

## API Routes

- `/api/leads`: Lead capture endpoint with storage utilities

## Environment Variables

Check `lib/env.ts` for environment variable validation and requirements.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nakoliss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
