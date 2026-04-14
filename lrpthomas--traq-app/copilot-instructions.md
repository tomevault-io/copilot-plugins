## traq-app

> Progressive Web App for Tree Risk Assessment Qualified (TRAQ) form completion based on the ISA Basic Tree Risk Assessment Form (2017). Supports PDF form filling, custom report generation, and offline-first operation.

# TRAQ Form App

## Overview
Progressive Web App for Tree Risk Assessment Qualified (TRAQ) form completion based on the ISA Basic Tree Risk Assessment Form (2017). Supports PDF form filling, custom report generation, and offline-first operation.

## Project Management System

This project uses a centralized management system in `.project/`:

```
.project/
├── PROJECT_STATUS.yaml    # Single source of truth - update this!
├── categories.yaml        # Feature categories and modules
├── schemas/
│   ├── supabase-schema.sql   # Database schema
│   └── rls-policies.sql      # Row Level Security policies
├── templates/
│   ├── ROADMAP.template.md   # Roadmap generator template
│   └── env.template          # Environment variables template
└── scripts/
    ├── generate-roadmap.js   # Generate ROADMAP.md from status
    ├── validate-status.js    # Validate PROJECT_STATUS.yaml
    └── sync-supabase-types.js # Generate TypeScript types from Supabase
```

### Project Commands
```bash
npm run project:status    # Validate PROJECT_STATUS.yaml
npm run project:roadmap   # Generate ROADMAP.md
npm run sync:types        # Generate Supabase TypeScript types
```

### Updating Project Status
1. Edit `.project/PROJECT_STATUS.yaml` to update features, phases, or blockers
2. Run `npm run project:status` to validate changes
3. Run `npm run project:roadmap` to regenerate ROADMAP.md

## Tech Stack
- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS + shadcn/ui
- **Database**: IndexedDB via Dexie.js (offline-first)
- **PDF**: pdf-lib (form filling), custom report generation
- **PWA**: Serwist (service worker)
- **Mobile**: Capacitor (iOS/Android wrapper)
- **Testing**: Vitest + React Testing Library
- **Validation**: Zod schemas

## Project Structure
```
src/
├── app/                          # Next.js App Router pages
│   ├── page.tsx                  # Home - assessment list
│   ├── layout.tsx                # Root layout with Header
│   ├── assessment/
│   │   ├── new/page.tsx          # New assessment
│   │   └── [id]/page.tsx         # Edit existing
│   └── settings/page.tsx         # App settings
├── components/
│   ├── ui/                       # shadcn/ui components
│   ├── form/
│   │   ├── AssessmentForm.tsx    # Main form with stepper
│   │   ├── FormField.tsx         # Field wrapper with tooltips
│   │   └── sections/             # Form section components
│   │       ├── HeaderSection.tsx
│   │       ├── TargetSection.tsx
│   │       ├── SiteFactorsSection.tsx
│   │       ├── TreeHealthSection.tsx
│   │       ├── LoadFactorsSection.tsx
│   │       ├── CrownBranchesSection.tsx
│   │       ├── TrunkSection.tsx
│   │       ├── RootsSection.tsx
│   │       ├── RiskCategorizationSection.tsx
│   │       └── MitigationSection.tsx
│   ├── layout/
│   │   └── Header.tsx            # App header with navigation
│   └── report/                   # Report components
├── lib/
│   ├── db/
│   │   └── index.ts              # Dexie database setup
│   ├── riskMatrix.ts             # Risk calculation matrices
│   ├── validation.ts             # Zod validation schemas
│   ├── fieldHelp.ts              # Tooltip/help content
│   ├── pdfGenerator.ts           # PDF generation
│   └── utils.ts                  # Utility functions
├── types/
│   ├── traq.ts                   # TypeScript interfaces (matches 2017 PDF)
│   └── species.ts                # Tree species types (auto-generated)
├── hooks/
│   ├── useAssessment.ts          # Assessment CRUD operations
│   ├── useMemory.ts              # Answer memory system
│   ├── useMedia.ts               # Media/photo handling
│   └── useTreeSpecies.ts         # Species data loading
├── test/
│   └── setup.ts                  # Vitest test setup
└── data/
    └── tree-species.csv          # Tree species dropdown data
```

## Commands

### Development
```bash
npm run dev              # Start dev server
npm run build            # Production build
npm run start            # Start production server
npm run lint             # ESLint
```

### Testing
```bash
npm run test             # Run tests in watch mode
npm run test:run         # Run tests once
npm run test:coverage    # Run tests with coverage report
```

### Native Apps (Capacitor)
```bash
npm run build:native     # Build for native (static export)
npm run native:ios       # Build and open iOS project
npm run native:android   # Build and open Android project
npm run cap:sync         # Sync web assets to native projects
```

### Data Utilities
```bash
npm run validate:data    # Validate tree-species.csv
npm run generate:types   # Generate species TypeScript types from CSV
```

## Form Structure (Matches ISA 2017 Form)

### Page 1 - Field Assessment
1. **Header**: Client, date, location, tree info, assessor
2. **Target Assessment**: Up to 4 targets with zones and occupancy
3. **Site Factors**: History, topography, soil, weather
4. **Tree Health**: Vigor, foliage, pests, species failure profile
5. **Load Factors**: Wind exposure, crown size/density
6. **Crown & Branches**: Defects, pruning history, failure assessment
7. **Trunk**: Defects, cavity, lean, failure assessment
8. **Roots & Root Collar**: Defects, failure assessment

### Page 2 - Risk Analysis
9. **Risk Categorization**: Multiple rows with Matrix 1 & 2 calculations
10. **Mitigation & Summary**: Options, residual risk, inspection interval

## Risk Matrix Logic
- **Matrix 1**: Likelihood of Failure × Likelihood of Impact → Failure & Impact
- **Matrix 2**: Failure & Impact × Consequences → Risk Rating
- **Overall Risk**: Highest individual risk rating from all rows

Key functions in `src/lib/riskMatrix.ts`:
- `calculateFailureAndImpact()` - Matrix 1 calculation
- `calculateRiskRating()` - Matrix 2 calculation
- `calculateFullRisk()` - Complete risk from all inputs
- `getOverallRiskRating()` - Get highest risk from array

## Validation

Use Zod schemas from `src/lib/validation.ts` for form data validation:

```typescript
import { assessmentSchema, validateHeaderInfo, getValidationErrors } from '@/lib/validation';

// Validate full assessment
const result = assessmentSchema.safeParse(data);
if (!result.success) {
  const errors = getValidationErrors(result.error);
}

// Validate specific sections
const headerResult = validateHeaderInfo(headerData);
```

Key validation helpers:
- `percentageSchema` - Validates 0-100 range
- `isValidPercentage()` - Quick percentage check
- `validateFoliagePercentages()` - Ensures foliage percentages sum ≤ 100

## Key Features
1. **Form Sections**: All ISA TRAQ form fields with tooltips/help from instructions
2. **Risk Matrices**: Auto-calculation with Matrix 1 & 2
3. **PDF Export**: Fill official TRAQ PDF form template
4. **Report Generation**: Custom PDF report
5. **Offline-First**: Full functionality without internet via IndexedDB
6. **Answer Memory**: Toggle to remember values per field
7. **Progressive Navigation**: Step-by-step form with progress indicator

## Database Tables (Dexie/IndexedDB)
- `assessments`: Complete TRAQ assessments
- `media`: Photos/attachments (as blobs)
- `memory`: Remembered field values per field path
- `settings`: App configuration

## Conventions
- Components: PascalCase (e.g., `TreeHealthSection.tsx`)
- Hooks: camelCase with `use` prefix (e.g., `useAssessment.ts`)
- Types: PascalCase matching ISA form terminology
- Tests: Co-located with source files (e.g., `riskMatrix.test.ts`)
- Use `cn()` utility for conditional classNames
- All form fields wrapped with `<FormField>` for tooltips
- Offline-first: Always save to IndexedDB
- Validate data with Zod schemas before critical operations

## Testing Patterns

### Unit Tests
Tests are co-located with source files. Use Vitest:

```typescript
import { describe, it, expect } from 'vitest'

describe('functionName', () => {
  it('should do something', () => {
    expect(result).toBe(expected)
  })
})
```

### Key Test Files
- `src/lib/riskMatrix.test.ts` - Risk calculation logic (55 tests)
- `src/lib/validation.test.ts` - Zod validation schemas (31 tests)

### Running Specific Tests
```bash
npm run test -- riskMatrix     # Run only riskMatrix tests
npm run test -- validation     # Run only validation tests
```

## PDF Template
Place the official ISA TRAQ PDF form at: `public/templates/traq-form.pdf`

The PDF field names are mapped in `src/lib/pdfGenerator.ts`. To debug field names,
temporarily enable console logging in the `generateFilledPDF` function.

## Code Quality

### Before Committing
1. Run tests: `npm run test:run`
2. Run linting: `npm run lint`
3. Validate data: `npm run validate:data`

### Type Safety
- TypeScript strict mode is enabled
- All assessment data types are defined in `src/types/traq.ts`
- Zod schemas provide runtime validation matching TypeScript types

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lrpthomas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lrpthomas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
