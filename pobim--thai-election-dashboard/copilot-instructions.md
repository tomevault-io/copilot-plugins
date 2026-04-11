## thai-election-dashboard

> **Thai Election Dashboard** - A Next.js dashboard visualizing Thailand's eligible voter data by region.

# AGENTS.md - Development Guide

## Project Overview

**Thai Election Dashboard** - A Next.js dashboard visualizing Thailand's eligible voter data by region.

## Build / Lint / Test Commands

```bash
# Development
npm run dev              # Start dev server at http://localhost:3000

# Production
npm run build            # Build for production
npm run start            # Start production server

# Linting
npm run lint             # Run ESLint (Next.js default config)
```

**No tests configured** - Add tests via Jest/Vitest if needed.

## Tech Stack

- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript 5.8 (strict mode enabled)
- **Styling:** Tailwind CSS 3.4
- **UI Components:** Custom + Tailwind
- **Charts:** Recharts
- **Maps:** react-simple-maps
- **Icons:** Lucide React
- **Font:** Kanit (Google Fonts) - Thai + Latin subsets

## Code Style Guidelines

### TypeScript
- Enable `strict: true` in tsconfig.json
- Use explicit return types on exported functions
- Prefer interfaces over type aliases for object shapes
- Use enums for fixed sets of string values (e.g., `RegionFilter`)

### File Structure
```
app/
├── components/          # React components (PascalCase)
├── types.ts            # Shared TypeScript types
├── globals.css         # Global styles + Tailwind
├── layout.tsx          # Root layout
└── page.tsx            # Main page (Home)
```

### Naming Conventions
- **Components:** PascalCase (e.g., `ThailandMap.tsx`, `StatsCard.tsx`)
- **Functions/Variables:** camelCase (e.g., `fetchElectionData`, `selectedRegion`)
- **Interfaces/Types:** PascalCase (e.g., `ElectionData`, `RegionFilter`)
- **Enums:** PascalCase with PascalCase keys (e.g., `RegionFilter.NORTH`)
- **Constants:** UPPER_SNAKE_CASE for true constants

### Imports Order
1. React imports
2. Third-party libraries (Next.js, etc.)
3. Local types
4. Local components
5. Local services/utils
6. Relative imports (last resort)

Example:
```typescript
import React, { useEffect, useState } from 'react';
import { ComposableMap } from 'react-simple-maps';
import { RegionData } from '../types';
import { StatsCard } from './components/StatsCard';
```

### Component Patterns
- Use `'use client'` for client components
- Prefer functional components with hooks
- Export components as named exports: `export const ComponentName`
- Use React.FC<Props> for type safety
- Destructure props in function parameters

### Styling
- Use Tailwind CSS utility classes exclusively
- Use arbitrary values sparingly: `h-[600px]`
- Prefer Tailwind's built-in color palette
- Custom colors via REGION_COLORS mapping for maps
- Font family via CSS variable: `font-sans` (Kanit)

### Error Handling
- Wrap async calls in try/catch
- Return fallback data on API failures to prevent UI crashes
- Use console.error for logging errors
- Display user-friendly error messages in Thai

### State Management
- Use React hooks (useState, useEffect, useMemo)
- Memoize expensive computations with useMemo
- Use functional updates for state setters

### Path Aliases
- Use `@/*` for root imports (e.g., `@/app/types`)
- Configured in tsconfig.json

### ESLint
- Uses `eslint-config-next` (Next.js default)
- Can disable specific rules with: `// eslint-disable-next-line rule-name`

## Notes

- Data is static mock data for demonstration
- Map uses external TopoJSON: `apisit/thailand.json`
- Supports Thai and English text
- Responsive design (mobile-first)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/POBIM)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/POBIM)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
