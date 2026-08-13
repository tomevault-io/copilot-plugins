## shopping-list

> This file provides context for LLMs and AI coding assistants working on this codebase.

# AGENTS.md - AI Agent Context

This file provides context for LLMs and AI coding assistants working on this codebase.

## Project Overview

**Purpose**: Shopping list web app replacing Google Spreadsheets workflow for monthly grocery management.

**Current State**: MVP complete with all 6 user stories implemented. No authentication, no persistence.

**Branch**: `001-mvp-shopping-list`

## Architecture

### Tech Stack
- **Next.js 16+** with App Router (not Pages Router)
- **TypeScript 5.x** with strict mode enabled
- **Tailwind CSS 4.x** with class-based dark mode
- **React 19** with hooks (useState, useCallback, useMemo)

### Directory Structure
```
src/
├── app/           # Next.js App Router pages
├── components/    # React components
├── data/          # Static data (products.ts)
├── hooks/         # Custom React hooks
└── types/         # TypeScript interfaces
```

### Key Files

| File | Purpose |
|------|---------|
| `src/types/product.ts` | Core types: `Product`, `ProductState`, `FilterType` |
| `src/data/products.ts` | Hardcoded product catalog (58 products) |
| `src/hooks/useProductState.ts` | State management: filters, toggles, quantities |
| `src/components/ProductRow.tsx` | Individual product row with all controls |
| `src/components/ProductList.tsx` | List container with empty state |
| `src/components/FilterBar.tsx` | Filter buttons with counts |
| `src/app/page.tsx` | Main page composing all components |
| `baselist.csv` | Source data for products (name, URL) |

## Data Model

```typescript
interface Product {
  id: string
  name: string
  url?: string           // Optional supermarket URL
  defaultQuantity: number
  defaultHay?: boolean   // Default "have it" state from spreadsheet
}

interface ProductState extends Product {
  quantity: number
  hay: boolean           // "Have it" - initialized from defaultHay
  comprado: boolean      // "Purchased" - added to cart
}

type FilterType = 'ALL' | 'PENDING' | 'MISSING'
```

## Filter Logic

- **ALL**: Show all products
- **PENDING**: `!hay` (needs to be purchased)
- **MISSING**: `!hay && !comprado` (couldn't buy at store)

## Code Conventions

### Naming
- Components: PascalCase (`ProductRow.tsx`)
- Hooks: camelCase with `use` prefix (`useProductState.ts`)
- Types: PascalCase (`ProductState`)
- Constants: SCREAMING_SNAKE_CASE (`FILTER_LABELS`)

### Component Pattern
- Use `'use client'` directive for interactive components
- Props interfaces defined above component
- Tailwind classes inline (no separate CSS modules)

### State Management
- Local state via `useState` in custom hook
- No external state library (Redux, Zustand)
- State resets on page refresh (intentional for MVP)

## Design Requirements

### From Constitution
- **Dark mode by default**: `html` element has `class="dark"`
- **Mobile-first**: Cards on mobile, table layout on desktop
- **Touch targets**: Minimum 44x44px for interactive elements
- **Performance**: FCP < 1.5s, bundle < 200KB gzipped

### CSS Variables (globals.css)
```css
--background, --foreground     /* Base colors */
--card-bg, --card-border       /* Card styling */
--primary, --primary-hover     /* Accent colors */
--muted                        /* Secondary text */
--success                      /* "Hay" checkbox */
```

## Common Tasks

### Adding a New Product
Edit `src/data/products.ts`:
```typescript
{
  id: 'unique-slug',
  name: 'Product Name',
  url: 'https://masonline.com.ar/...', // optional
  defaultQuantity: 0,
  defaultHay: false  // optional, defaults to false
}
```

### Modifying Filter Logic
Edit `useProductState.ts` → `filteredProducts` useMemo

### Changing Visual Styles
Edit component Tailwind classes directly or `globals.css` for CSS variables

### Adding New Filter Type
1. Add to `FilterType` in `types/product.ts`
2. Add label to `FILTER_LABELS`
3. Add case in `useProductState.ts` → `filteredProducts`
4. FilterBar will automatically render new button

## Specifications

Full feature specification: `specs/001-mvp-shopping-list/spec.md`
Implementation plan: `specs/001-mvp-shopping-list/plan.md`
Task breakdown: `specs/001-mvp-shopping-list/tasks.md`

## Future Iterations (Not in MVP)

- Firebase Authentication
- Firestore persistence
- Receipt upload with OCR
- Price tracking and inflation history
- MCP integration for AI assistants

## Commands

```bash
npm run dev      # Development server (localhost:3000)
npm run build    # Production build
npm run start    # Production server
npm run lint     # ESLint check
```

---
> Source: [joelmarquez90/shopping-list](https://github.com/joelmarquez90/shopping-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
