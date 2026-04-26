## nyc-lens

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BBL Club is a Next.js 16 application for exploring NYC real estate data (ACRIS, PLUTO, DOB, HPD datasets). Built with React 19, TypeScript, Tailwind CSS v4, and ag-Grid Enterprise for complex data tables with server-side filtering/sorting via Elasticsearch.

## Development Commands

### Core Development
- `npm run dev` - Start development server with Turbopack (http://localhost:3000)
- `npm run build` - Production build (uses webpack; Turbopack is dev-only in Next.js 16)
- `npm start` - Start production server
- `npm run lint` - Run ESLint

### Testing
- `npm test` - Run all tests (unit + Storybook) with Vitest
- `npm run test:watch` - Run tests in watch mode
- Single test file: `npx vitest run path/to/test.test.tsx`
- Test a specific component: `npx vitest run -t "component name"`

### Storybook
- `npm run storybook` - Start Storybook dev server on port 6006
- `npm run build-storybook` - Build static Storybook

## Architecture Overview

### Application Structure

```
src/
├── app/                        # Next.js App Router pages
│   ├── api/                    # API routes (server-side only)
│   │   └── acris/              # ACRIS data endpoints (properties, documents, parties)
│   ├── property/[bbl]/         # Property detail pages (dynamic BBL routes)
│   │   ├── dob/                # DOB data tabs (violations, permits, jobs, etc.)
│   │   ├── hpd/                # HPD data tabs (violations, registrations, permits)
│   │   ├── pluto/              # PLUTO property data
│   │   └── tax/                # Tax assessment data
│   ├── dob/                    # DOB dataset exploration pages
│   ├── hpd/                    # HPD dataset exploration pages
│   └── search/                 # Property search interface
├── components/
│   ├── ui/                     # Reusable primitives (Button, Tabs, Switch, Autocomplete)
│   ├── table/                  # ag-Grid table components (property, document, party)
│   ├── search/                 # Search-specific components (PropertyAutocomplete, HighlightedText)
│   ├── layout/                 # Layout components (ResizableSidebarLayout, SidebarNav)
│   └── icons/                  # Icon components
├── data/                       # Data access layer (server-only)
│   ├── elasticsearch.ts        # Elasticsearch client
│   ├── db.ts                   # MSSQL database connection
│   ├── pluto.ts                # PLUTO data queries
│   ├── valuation.ts            # Tax valuation queries
│   ├── dobJobs.ts              # DOB jobs data
│   └── dobViolations.ts        # DOB violations data
├── types/                      # TypeScript type definitions
│   ├── acris.ts                # ACRIS record types
│   ├── pluto.ts                # PLUTO field types
│   ├── dob.ts                  # DOB data types
│   ├── valuation.ts            # Tax assessment types
│   └── api.ts                  # API request/response types
├── utils/                      # Utility functions
│   ├── cn.ts                   # Tailwind class name merger (clsx + tailwind-merge)
│   ├── agGrid.ts               # ag-Grid ↔ Elasticsearch query builder
│   └── formatters.ts           # Data formatting utilities
└── constants/                  # Constants and lookup tables
    ├── acris.ts                # ACRIS document types, control codes
    ├── nyc.ts                  # Borough codes, BBL utilities
    └── landUse.ts              # Land use classifications
```

### Key Architectural Patterns

#### Server-Side Data Access
- **All data queries are server-side only** - Files in `src/data/` use `'server-only'` to prevent client bundling
- **Two data sources**:
  - Elasticsearch: ACRIS property records (properties, documents, parties)
  - MSSQL: NYC Open Data (PLUTO, tax assessments, DOB, HPD)
- **API routes** in `src/app/api/acris/` handle ag-Grid server-side requests, converting ag-Grid filter/sort models to Elasticsearch DSL via `buildEsQueryFromAgGrid()` in `src/utils/agGrid.ts`

#### ag-Grid Server-Side Model
- **PropertyTable** (`src/components/table/property/PropertyTable.tsx`): Main ACRIS property table
  - Uses `serverSideDatasource` to fetch data via `/api/acris/properties`
  - Supports filtering, sorting, pagination (10K row limit)
  - Master-detail rows expand to show related documents (DocumentTable)
- **DocumentTable**: Nested table showing ACRIS documents for a property
  - Fetches data via `/api/acris/documents`
  - Also uses server-side model
- **ag-Grid Enterprise license**: Required (configured in `PropertyTable.tsx:14`)

#### Next.js App Router Patterns
- **Server Components by default**: Use `'use client'` only when needed (hooks, events, browser APIs)
- **Dynamic routes**: `app/property/[bbl]/` uses BBL (Borough-Block-Lot) as route param
- **Parallel routes**: Property pages use tabs for different data sources (PLUTO, DOB, HPD, etc.)
- **Data fetching**: Server Components call `src/data/` functions directly; Client Components use API routes

#### Property Detail Pages (`app/property/[bbl]/`)
- **BBL format**: 10-digit string (1 borough + 5 block + 4 lot), e.g., `"1000010001"`
- **Tab-based navigation**: Each data source (PLUTO, Tax, DOB, HPD) has its own page under `[bbl]/`
- **Data display pattern**: Fetch data in page, pass to display components
  - Example: `app/property/[bbl]/pluto/page.tsx` fetches PLUTO data, renders `PlutoTabDisplay`
- **DOB/HPD sub-tabs**: Use `DobTabNav` for nested navigation (violations, permits, jobs, etc.)

#### Search & Autocomplete
- **PropertyAutocomplete** (location TBD): Built on generic `Autocomplete` component (`src/components/ui/Autocomplete.tsx`)
- Uses `@algolia/autocomplete-core` for flexible autocomplete behavior
- **Text matching**: `src/components/search/textMatcher.ts` with synonym support (`synonyms.json`)
- **Highlighted results**: `HighlightedText` component highlights matched text

#### Parcel Map
- **ParcelMap** (`src/components/map/ParcelMap.tsx`): Interactive Mapbox map showing NYC parcel boundaries
- Uses Mapbox GL JS tileset (`svayser.nys-tax-parcels-prod`) with vector tiles
- **BBL utilities** (`src/utils/bbl.ts`): Converts between URL format (4-476-1) and SBL format (4004760001)
- **Features**:
  - Highlights selected parcel in amber
  - Hover effects on parcels (blue highlight)
  - Click to navigate to property overview
  - Fullscreen modal dialog mode
  - Satellite view with parcel overlays
- **Integration**: Used in property overview page, centered on PLUTO coordinates

#### Styling & Design System
- **Tailwind v4** with tokens in `src/app/globals.css` (see `@theme inline`)
- **Design tokens**: Use `bg-background`, `text-foreground` (auto dark mode via `prefers-color-scheme`)
- **Class merging**: Use `cn()` utility (`src/utils/cn.ts`) for conditional classes
- **Detailed standards**: See `.cursor/rules/css-and-markup-standards.mdc` for comprehensive component patterns, accessibility requirements, and class ordering conventions
- **ag-Grid theming**: Custom theme in `src/components/table/theme.ts`, overrides in `globals.css` scoped to `.ag-theme-*`

#### Testing Strategy
- **Vitest** with two projects:
  1. Unit tests (`src/**/*.test.{ts,tsx}`) - happy-dom environment
  2. Storybook tests - Playwright browser tests via `@storybook/addon-vitest`
- **Test setup**: `vitest.setup.ts` for global config
- **Path aliases**: `@/` maps to `src/` (configured in `vitest.config.ts`)

## Environment Variables

Required in `.env.local`:

```bash
# Elasticsearch (ACRIS data)
ELASTICSEARCH_NODE=https://...
ELASTICSEARCH_USERNAME=...
ELASTICSEARCH_PASSWORD=...
ELASTICSEARCH_ACRIS_INDEX_NAME=...

# MSSQL (NYC Open Data)
DB_SERVER=...
DB_PORT=...
DB_USER=...
DB_PASSWORD=...
DB_DATABASE=...

# Mapbox (Parcel Map)
NEXT_PUBLIC_MAPBOX_ACCESS_TOKEN=pk.eyJ1Ijoi...

# Clerk (Authentication)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=YOUR_PUBLISHABLE_KEY
CLERK_SECRET_KEY=YOUR_SECRET_KEY
```

## Important Implementation Notes

### Working with ag-Grid
1. **Filter/sort translation**: `buildEsQueryFromAgGrid()` in `src/utils/agGrid.ts` converts ag-Grid models to Elasticsearch DSL
   - Handles text, number, date, set filters
   - Maps field types using embedded Elasticsearch mapping (see `elasticMapping` at bottom of file)
   - Resolves sortable fields (e.g., `block` → `block.integer` for sorting text fields)
2. **Column definitions**: See `src/components/table/property/columnDefs.ts` for property columns
3. **Pagination limit**: Hard-coded 10K row limit (Elasticsearch `max_result_window`)
4. **Menu customization**: `src/components/table/constants/menu.ts` defines column menu items

### Working with BBL (Borough-Block-Lot)
- **Format**: 10-digit string (no hyphens) - `"1000010001"` = Manhattan, Block 1, Lot 1
- **Utilities**: `src/constants/nyc.ts` has borough lookup functions
- **Elasticsearch mapping**: BBL components stored as `borough`, `block.integer`, `lot.integer`

### Server-Side Data Fetching
- **MSSQL queries**: Use `executeQuery()` from `src/data/db.ts`
- **Elasticsearch queries**: Use `search()` from `src/data/elasticsearch.ts`
- **Caching**: Next.js automatically caches Server Component fetches; use `cache: 'no-store'` or revalidation if needed
- **Error handling**: Wrap data fetches in try-catch, return error states to display components

### Adding New Components
- **Location**:
  - Shared primitives → `src/components/ui/`
  - Feature-specific → `src/components/<feature>/`
  - Route-specific → `src/app/<route>/components/`
- **Follow patterns** in `.cursor/rules/css-and-markup-standards.mdc`:
  - Server Component by default
  - Accept `className` prop for composition
  - Use design tokens (`bg-background`, `text-foreground`)
  - Extend native HTML props (`ComponentPropsWithoutRef<'button'>`)
- **Storybook stories**: Create `*.stories.tsx` files for visual testing
- **Tests**: Create `__tests__/*.test.tsx` files alongside components

### Working with NYC Open Data
- **PLUTO**: NYC property data (lot size, building class, zoning) - see `src/types/pluto.ts` for 200+ fields
- **DOB**: Department of Buildings (violations, permits, jobs, complaints)
- **HPD**: Housing Preservation & Development (violations, registrations)
- **ACRIS**: Automated City Register Information System (property sales, mortgages, deeds)
- **Field descriptions**: Many datasets have embedded field metadata (e.g., `src/app/property/[bbl]/pluto/metadata.json`)

### Next.js Server Components Best Practices
- **Data fetching**: Fetch in Server Components, pass props to Client Components
- **Database calls**: Never import `src/data/db.ts` or `src/data/elasticsearch.ts` in Client Components (protected by `'server-only'`)
- **API routes**: Use for client-side data fetching (e.g., ag-Grid datasources)
- **Streaming**: Use `<Suspense>` boundaries for progressive rendering of slow queries

## Common Pitfalls

1. **Turbopack for production**: NEVER use `--turbopack` flag with `npm run build` - it's dev-only and will cause deployment failures (missing routes-manifest.json). Only use Turbopack for `npm run dev`.
2. **ag-Grid license**: Ensure license key is valid (expires July 2026)
3. **Elasticsearch field types**: Text fields need `.keyword` or `.integer` subfields for exact matching/sorting - see `searchStrategies` in `src/utils/agGrid.ts`
4. **BBL format**: Always use 10-digit string format, never numeric (leading zeros matter)
5. **Server/Client boundary**: Don't accidentally import server-only modules in client components
6. **Tailwind purging**: Never use dynamic class construction like `text-${size}` - use complete class names
7. **Date handling**: Elasticsearch expects ISO 8601 format - use `normalizeToIsoDateTime()` in `src/utils/agGrid.ts`
8. **Vitest timezone**: Tests run in UTC (configured in `vitest.config.ts`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LegendariTech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
