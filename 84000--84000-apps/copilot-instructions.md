## 84000-apps

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
# Install dependencies (run from repo root)
npm install

# Run an app in dev mode
npx nx dev web-main          # Main app (Studio)
npx nx dev web-editor        # Editor app
npx nx dev web-reader        # Reader app
npx nx dev web-docs          # Docs app

# Build for production
npx nx build web-main        # Build a specific app
npx nx build <project-name>  # Build any project

# Run tests
npx nx test <project-name>   # Run tests for a specific project
npx nx test design-system    # Example: test design-system

# Lint
npx nx lint <project-name>   # Lint a specific project
npm run lint:css             # Lint all CSS files
npm run lint:css:fix         # Auto-fix CSS lint issues

# Storybook (design system)
npx nx storybook design-system

# Run multiple targets
npx nx run-many -t build lint -p web-main design-system  # Multiple projects and targets
```

## Architecture Overview

This is an **Nx monorepo** containing multiple Next.js applications and shared libraries. Code should be placed in `libs/` and reused across `apps/`. Applications should be thin orchestration layers.

### Apps (`/apps`)

- **web-main**: Main web app (Studio) - Next.js with TipTap collaborative editor
- **web-editor**: Editor application
- **web-reader**: Reader application
- **web-reader-mfe**: Reader micro-frontend
- **web-docs**: Documentation site (Nextra)
- **node-scripts**: Node.js utility scripts

### Libraries (`/libs`)

**Core libraries:**

- **data-access**: Supabase client and data layer (auth, canon, glossary, publications, storage, etc.). Has `/ssr` export for server-side use.
- **design-system**: Shared UI components (Button, Dialog, Card, Table, etc.) built with Radix UI primitives. Run Storybook to explore.
- **lib-utils**: Utility functions (compare, style helpers, highlight, hooks, string manipulation)

**Feature libraries:**

- **lib-editing**: TipTap editor components, block types, passage and title editing
- **lib-user**: User authentication components (Login, ProfileDropdown, SessionContext, Library pages)
- **lib-canon**: Canon navigation context, sidebar, and page components
- **lib-glossary**: Glossary editor and tables
- **lib-search**: Search data layer and UI components
- **lib-explore**: Explore feature configuration and pages
- **lib-instr**: Instrumentation/analytics (PostHog integration, feature flags)

### Path Aliases

Import libraries using aliases defined in `tsconfig.base.json`:

```typescript
import { Button } from '@eightyfourthousand/design-system';
import { createBrowserClient } from '@eightyfourthousand/data-access';
import { createServerClient } from '@eightyfourthousand/data-access/ssr';
import { useProfile } from '@lib-user';
import { GlossaryEditor } from '@lib-glossary';
```

Many libraries have `/ssr` exports for server-side components.

## Key Technologies

- **Next.js 16** with React 19
- **TipTap** for collaborative rich text editing
- **Supabase** for backend (auth, database, storage)
- **Radix UI** primitives for accessible components
- **Tailwind CSS 4** for styling
- **PostHog** for analytics and feature flags
- **Jest** for unit testing
- **Storybook 10** for component development

## Environment Setup

For Vercel-deployed apps, pull environment variables from within the app directory:

```bash
cd apps/web-main
npx vercel env pull --environment development
```

This creates `.env.local` with required variables.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/84000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/84000)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
