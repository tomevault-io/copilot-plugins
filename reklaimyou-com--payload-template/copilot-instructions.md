## payload-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 16 + PayloadCMS 3 template for building headless CMS-powered websites. It uses React 19, TypeScript, TailwindCSS 4, and PostgreSQL.

## Commands

### Development
```bash
pnpm dev              # Start development server (localhost:3000)
pnpm devsafe          # Clear .next cache and start fresh
```

### Building & Production
```bash
pnpm build            # Production build
pnpm start            # Start production server
pnpm next:analyze     # Analyze bundle size
```

### Code Quality
```bash
pnpm lint             # Run Biome linter
pnpm format           # Format code with Ultracite
pnpm check            # Biome check with auto-fixes
```

### Testing
```bash
pnpm test             # Run unit tests once
pnpm test:watch       # Watch mode
pnpm test:ui          # Vitest UI dashboard
pnpm test:coverage    # Generate coverage reports
pnpm e2e              # Run Playwright E2E tests
pnpm e2e:ui           # Playwright UI mode
pnpm e2e:headed       # Run E2E with browser visible
```

### PayloadCMS
```bash
pnpm payload generate:types      # Regenerate TypeScript types after schema changes
pnpm payload generate:importmap  # Regenerate import map
pnpm payload migrate             # Run database migrations
pnpm payload seed                # Seed database with sample data
```

## Architecture

### Directory Structure

- **`app/(frontend)/`** - Public website routes (Next.js App Router)
- **`app/(payload)/`** - PayloadCMS admin panel (mounted at `/admin`)
- **`app/api/v1/`** - REST API endpoints (leads, preview)
- **`payload/`** - CMS configuration (blocks, collections, globals, hooks, fields)
- **`payload-config/`** - Modular Payload configuration (18 separate modules)
- **`components/`** - React components (ui/, blocks/, layout/, admin/)
- **`queries/`** - Cached data fetching functions with cache tags
- **`lib/`** - Utilities (env validation, URL handling, access control)
- **`tests/`** - Unit tests (Vitest)
- **`e2e/`** - E2E tests (Playwright)

### Key Patterns

**Data Fetching**: Queries use Next.js `unstable_cache()` with cache tags for invalidation. Retry logic with exponential backoff handles transient errors.

**Block-Based Pages**: 21+ reusable blocks in `/payload/blocks` are rendered dynamically with React.lazy() and error boundaries.

**Type Generation**: After modifying PayloadCMS collections or fields, run `pnpm payload generate:types` to update `payload-types.ts`.

**Path Aliases**: Use `@/*` for root imports and `@payload-config` for payload.config.ts.

### Configuration Files

- **`payload.config.ts`** - Main CMS config (imports from `/payload-config`)
- **`biome.jsonc`** - Linting rules (extends Ultracite presets)
- **`next.config.ts`** - Next.js config with security headers and Turbopack

### Environment Variables

Required in `.env.local`:
```
DATABASE_URI=postgresql://...
PAYLOAD_SECRET=<32+ chars>
PREVIEW_SECRET=<any>
```

Optional:
```
BLOB_READ_WRITE_TOKEN     # Vercel Blob storage
RESEND_API_KEY            # Email service
AUTOLOGIN=true            # Dev auto-login to admin
```

## Linting Rules

Biome enforces:
- No `console.log` statements
- No `any` types
- Use `for...of` over `.forEach()`
- Arrow functions for callbacks
- Next.js `<Image>` component required for images

Import order: Types → Node → npm → React/Next → Payload → Others → Relative → CSS → Assets

---
> Source: [ReklaimYou-com/payload-template](https://github.com/ReklaimYou-com/payload-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
