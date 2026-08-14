## aviemore-pet-services

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js landing page for Aviemore Pet Services, a pet care business offering dog walking, pet sitting, and pet taxi services in Aviemore, Scotland. Built from a Next.js Landing Page Starter Template with TypeScript, Tailwind CSS, and strict ESLint/Prettier configurations.

## Development Commands

### Essential Commands
- `npm run dev` - Start development server (http://localhost:3000)
- `npm run build` - Create production build
- `npm run start` - Start production server (run after `build`)
- `npm run lint` - Run ESLint on all files
- `npm run format` - Auto-fix ESLint issues and format JSON/YAML files
- `npm run check-types` - Run TypeScript type checking without emitting files

### Additional Commands
- `npm run build-stats` - Build with bundle analyzer (sets ANALYZE=true)
- `npm run build-prod` - Clean and build for production
- `npm run clean` - Remove .next, .swc, and out directories

## Architecture

### Component Structure

The codebase follows an atomic design pattern with clear separation of concerns:

1. **Entry Point**: `src/pages/index.tsx` → imports `Base` template
2. **Main Template**: `src/templates/Base.tsx` → orchestrates all template components
3. **Template Components**: `src/templates/*` → high-level sections (Hero, Banner, Footer, etc.)
4. **Atomic Components**: `src/*` folders → reusable UI elements used by templates

**Component Hierarchy:**
```
pages/index.tsx
└── templates/Base.tsx
    ├── layout/Meta.tsx (SEO metadata)
    ├── templates/Hero.tsx
    ├── templates/Reviews.tsx
    ├── templates/VerticalFeatures.tsx
    ├── templates/Banner.tsx
    └── templates/Footer.tsx
```

### Directory Structure

- `src/pages/` - Next.js pages and API routes
  - `src/pages/api/send-email.ts` - Contact form API endpoint using Resend
- `src/templates/` - High-level template components
- `src/layout/` - Layout components (Meta, Section)
- `src/background/`, `src/button/`, `src/cta/`, `src/feature/`, `src/footer/`, `src/hero/`, `src/navigation/` - Atomic components organized by function
- `src/utils/AppConfig.ts` - Centralized site configuration
- `public/assets/images/` - Static image assets

### Key Configuration Files

**TypeScript Path Aliases:**
- `@/*` maps to `./src/*`
- `@/public/*` maps to `./public/*`

**Next.js Configuration:**
- Bundle analyzer enabled with `ANALYZE=true` environment variable
- `trailingSlash: true` - URLs end with trailing slashes
- `reactStrictMode: true` - Strict mode enabled
- `poweredByHeader: false` - X-Powered-By header removed

**Custom Theme:**
- Primary brand color: Green (`#7CB342` - primary-500)
- Extended color palette defined in `tailwind.config.js`

### Environment Variables

Required for contact form functionality:
- `RESEND_API_KEY` - API key for Resend email service
- `RESEND_DELIVERY_EMAIL` - Destination email for contact form submissions

### ESLint and Code Quality

Strict ESLint configuration with:
- Airbnb TypeScript style guide
- Tailwind CSS linting
- Import sorting (simple-import-sort)
- Unused imports detection
- Prettier integration

**Important ESLint Rules:**
- Single quotes enforced
- Import type declarations required where necessary
- Named exports preferred over default exports
- Unused imports are errors

**Lint-Staged:**
Pre-commit hooks run on staged files:
- ESLint auto-fix and validation on `.ts(x)` and `.js(x)` files
- TypeScript type checking on all TypeScript files
- Prettier formatting on JSON files

### TypeScript Configuration

Extremely strict TypeScript settings enabled:
- `strict: true`
- `noUncheckedIndexedAccess: true`
- `noImplicitAny: true`
- `noImplicitReturns: true`
- `noUnusedLocals: true`
- `noUnusedParameters: true`

When adding new code, ensure it passes all strict TypeScript checks.

## Working with This Codebase

### Adding New Sections
1. Create atomic components in appropriate `src/*/` directories
2. Create template component in `src/templates/`
3. Import and add to `src/templates/Base.tsx`

### Modifying Site Content
- Update `src/utils/AppConfig.ts` for site metadata
- Template components in `src/templates/` contain page section content
- Images stored in `public/assets/images/`

### Styling
- Tailwind CSS utility classes throughout
- Custom theme colors defined in `tailwind.config.js`
- PostCSS configured for processing Tailwind with styled-jsx

### API Routes
Contact form submission handled by `src/pages/api/send-email.ts` using the Resend service. Expects POST requests with `email` and `message` fields.

## Pre-commit Workflow

Husky is configured with lint-staged:
1. Staged TypeScript/JavaScript files are linted and auto-fixed
2. Full TypeScript type check runs across entire codebase
3. JSON files are formatted with Prettier

If any step fails, the commit is blocked.

---
> Source: [WhiskyToad/aviemore-pet-services](https://github.com/WhiskyToad/aviemore-pet-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
