## nextjs-pocket-heist

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pocket Heist is a Next.js starter project for the Claude Code Masterclass. It's a task/mission management app with an office-themed "heist" concept. Currently in scaffold stage with page structure in place but core functionality not yet implemented.

## Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:3000)
npm run build        # Production build
npm start            # Start production server
npm run lint         # Run ESLint
npm test             # Run all Vitest tests
npm test -- <path>   # Run specific test file
```

## Architecture

### Route Structure

The app uses **Next.js App Router with route groups** to organize pages by access level without affecting URLs:

- **`app/(public)/`** - Public pages (no Navbar): landing (`/`), login, signup
- **`app/(dashboard)/`** - Protected pages (with Navbar): heists dashboard, create, details

Route groups `(public)` and `(dashboard)` don't appear in URLs but allow different layouts:
- `app/(public)/layout.tsx` - Simple layout for unauthenticated pages
- `app/(dashboard)/layout.tsx` - Includes `<Navbar />` component for authenticated area

### Component Pattern

Components follow a barrel export pattern with CSS Modules:
```
components/
└── ComponentName/
    ├── ComponentName.tsx
    ├── ComponentName.module.css
    └── index.ts              # Barrel export
```

Import using: `import ComponentName from "@/components/ComponentName"`

### Path Aliases

TypeScript is configured with `@/*` alias pointing to project root (see `tsconfig.json`). Always use this for imports:
- ✅ `import Navbar from "@/components/Navbar"`
- ❌ `import Navbar from "../../components/Navbar"`

### Styling

**Theme System**: Custom colors defined in `app/globals.css` using Tailwind v4's `@theme` directive:
- `--color-primary`, `--color-secondary` (purple/pink brand colors)
- `--color-dark`, `--color-light`, `--color-lighter` (backgrounds)
- `--color-success`, `--color-error` (status colors)
- `--color-heading`, `--color-body` (text colors)

**Utility Classes**: Global CSS defines reusable classes like `.page-content`, `.center-content`, `.form-title`

**Component Styles**: Use CSS Modules (`.module.css`) for component-specific styling

### Testing

Tests use **Vitest + React Testing Library**:
- Config: `vitest.config.mts` (jsdom environment, globals enabled)
- Setup: `vitest.setup.ts` loads `@testing-library/jest-dom`
- Location: `tests/` directory mirrors source structure
- Pattern: See `tests/components/Navbar.test.tsx` for reference

Import pattern (globals enabled in tsconfig):
```tsx
import { render, screen } from "@testing-library/react"
import { describe, it, expect } from "vitest"
```

## Current Implementation Status

This is a **starter project** with scaffolding only:
- Pages are placeholders with no functionality
- No state management implemented
- No API routes or backend
- No authentication logic
- Components are presentational only

When implementing features, you'll need to add these missing pieces.

---
> Source: [angad94-14/nextjs-pocket-heist](https://github.com/angad94-14/nextjs-pocket-heist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
