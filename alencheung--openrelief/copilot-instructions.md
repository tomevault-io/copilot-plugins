## openrelief

> npm run dev          # Start Next.js dev server

# AGENTS.md - Coding Agent Guidelines for OpenRelief

## Build/Lint/Test Commands

### Development

```bash
npm run dev          # Start Next.js dev server
npm run build        # Production build
npm run start        # Start production server
npm run lint         # Run ESLint
npm run lint:fix     # Fix linting issues
npm run type-check   # TypeScript type check (tsc --noEmit)
```

### Testing

```bash
npm run test                          # Run all Jest tests
npm run test:watch                    # Watch mode
npm run test:coverage                 # Coverage report

# Single test file
npx jest path/to/file.test.ts

# Single test with pattern
npx jest --testNamePattern="should add event"

# Specific test categories
npm run test:emergency                # Emergency-related tests
npm run test:trust                    # Trust system tests
npm run test:consensus                # Consensus engine tests
npm run test:hooks                    # useTrustSystem hook tests
npm run test:integration              # Integration tests
npm run test:spatial                  # Spatial query tests
npm run test:security                 # Security tests (node script)
npm run test:pwa                      # PWA tests (node script)

# E2E Tests
npm run test:e2e                      # Cypress tests
npm run test:e2e:open                 # Cypress UI mode
npm run test:e2e:playwright           # Playwright tests
npm run test:e2e:playwright:open      # Playwright UI mode
npm run test:e2e:playwright:debug     # Playwright debug mode

# Lighthouse / Performance
npm run test:lighthouse               # Run Lighthouse CI
npm run test:lighthouse:mobile        # Mobile Lighthouse
npm run test:lighthouse:desktop       # Desktop Lighthouse
npm run test:lighthouse:pwa           # PWA-focused Lighthouse
```

### Database (Supabase)

```bash
npm run db:generate   # Generate TypeScript types from schema
npm run db:migrate    # Push migrations
npm run db:reset      # Reset local database
npm run db:seed       # Seed local database
npm run supabase:start
npm run supabase:stop
```

### Formatting

```bash
npm run format        # Format with Prettier
npm run format:check  # Check formatting
```

## Project Architecture

- **Framework**: Next.js 15 (App Router) + React 18
- **Language**: TypeScript (strict mode, all strict checks enabled)
- **Database/Auth**: Supabase (Postgres, Auth, RLS, Realtime)
- **State**: Zustand (with persist + subscribeWithSelector middleware)
- **Data Fetching**: TanStack Query v5
- **Styling**: Tailwind CSS + CVA (class-variance-authority) + Radix UI
- **Maps**: MapLibre GL + Leaflet (dual map support)
- **Spatial**: Turf.js + geolib
- **Edge Functions**: Cloudflare Workers (see `src/edge/`)
- **Monitoring**: Sentry (client/server/edge)
- **Rate Limiting**: Upstash Redis
- **Validation**: Zod (runtime) + custom validators (`src/lib/validation.ts`)

## Code Style Guidelines

### Imports

Use path aliases defined in tsconfig: `@/*` maps to `./src/*`. Group: React/Next
first, external libs second, internal aliases third.

```typescript
import { useState, useEffect } from 'react'
import { useQuery, useMutation } from '@tanstack/react-query'
import { supabase } from '@/lib/supabase'
import { Database } from '@/types/database'
```

### Formatting (Prettier + ESLint)

- No semicolons
- Single quotes, avoid escapes only
- 2-space indentation
- No trailing commas (`trailingComma: "none"`)
- Max line: 100 chars (Prettier), 120 chars (ESLint warning)
- Curly braces required for all control structures
- Arrow functions: avoid parens for single param `x => x`
- Use `import type` for type-only imports
- `no-console`: warn (allow `console.warn`, `console.error`)
- `eqeqeq: ["error", "always"]` — always use `===`
- `prefer-const` is off — `let` is acceptable

### TypeScript

- Strict mode with all strict checks + `noUncheckedIndexedAccess`
- `noImplicitReturns`, `noFallthroughCasesInSwitch`, `noImplicitOverride`
- `exactOptionalPropertyTypes: true` — use `?` or `| undefined`, not both
- Use Database types from `@/types/database` for Supabase tables
- `@typescript-eslint/no-explicit-any` is off (allowed), but prefer typed
  alternatives
- Unused vars: prefix with `_` to suppress warnings

### Naming Conventions

- Components: PascalCase (`TrustBadge.tsx`, `EmergencyMap.tsx`)
- Hooks: camelCase with `use` prefix (`useEmergencyEvents.ts`)
- Stores: camelCase with `Store` suffix (`emergencyStore.ts`)
- Types/Interfaces: PascalCase (`EmergencyEvent`, `EmergencyFilter`)
- Utility files: kebab-case (`map-utils.ts`, `errorHandling.ts`)
- App Router: folder-based routing under `src/app/`

### React Components

- Function components with arrow functions
- Forward refs for UI primitives:

```typescript
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, ...props }, ref) => {
    return <Comp className={cn(buttonVariants({ variant, className }))} ref={ref} {...props} />
  }
)
Button.displayName = 'Button'
```

- Use CVA for component variants (see `src/components/ui/Button.tsx`)
- Merge classNames with `cn()` from `@/lib/utils` (clsx + tailwind-merge)
- Extract complex logic to custom hooks in `src/hooks/`
- Wrap pages with `<Providers>` (includes QueryClientProvider, etc.)

### State Management (Zustand)

```typescript
export const useEmergencyStore = create<EmergencyState>()(
  subscribeWithSelector(
    persist(
      (set, get) => ({ ...state, ...actions }),
      { name: 'emergency-store', partialize: (state) => ({...}) }
    )
  )
)
```

- Separate State and Actions interfaces
- Export selectors:
  `export const useEvents = () => useStore(state => state.events)`
- Store files in `src/store/`

### Data Fetching (TanStack Query)

- Query hooks for reads, mutation hooks for writes (`src/hooks/queries/`)
- Query keys as arrays: `['emergency-events', filters]`
- Invalidate on mutations:
  `queryClient.invalidateQueries({ queryKey: ['emergency-events'] })`
- Set `enabled` flag for conditional queries

### Error Handling

- Structured error classification: `@/lib/errorHandling.ts`
- ErrorInfo type with severity levels: `low | medium | high | critical`
- Retry with exponential backoff via `RetryConfig`
- Always provide user-facing error messages
- Log errors with context; never expose secrets

### Validation

- Use Zod schemas for runtime validation of user input
- Custom validators in `src/lib/validation.ts` for form fields
- Sanitize HTML with `isomorphic-dompurify`

### Testing

- Jest with next/jest, jsdom environment
- Tests in `__tests__/` dirs or `.test.ts`/`.spec.ts` suffix
- Use `@testing-library/react` + `@testing-library/jest-dom`
- `data-testid` attribute for queries (configured in jest.setup.js)
- Global mocks in `jest.setup.js`: router, image, fetch, Supabase, TanStack
  Query, MapLibre, Leaflet, localStorage, geolocation, IntersectionObserver
- Fixtures from `@/test-utils/fixtures/`
- Custom test utils from `@/test-utils/`
- Reset stores in beforeEach: `const { reset } = useStore.getState(); reset()`
- Coverage thresholds: 70% global, 85% for map components, 90% for supabase
  client

### Security

- Never log or commit secrets/API keys
- Next.js middleware enforces security headers, rate limiting, input validation
  (`src/middleware.ts`)
- Supabase RLS policies for data access control
- Redis-backed rate limiting via Upstash
- Trust-based security middleware for API routes
- Sanitize HTML with `isomorphic-dompurify`

### Edge Functions

- Cloudflare Workers in `src/edge/`
- Config in `wrangler.toml` (dev) and `wrangler.production.toml`

### File Organization

- `src/app/` — Next.js App Router pages and API routes
- `src/components/` — React components organized by feature (ui/, map/, trust/,
  emergency/, etc.)
- `src/hooks/` — Custom React hooks
- `src/store/` — Zustand stores
- `src/lib/` — Utilities, configs, error handling, monitoring, security
- `src/types/` — TypeScript type definitions
- `src/edge/` — Cloudflare Workers
- `src/test-utils/` — Test helpers and fixtures
- One component per file, export from barrel `index.ts`
- Keep files under 500 lines

## Pre-commit Hooks

- Husky + lint-staged runs on commit
- Auto-fixes ESLint issues and formats with Prettier
- Blocks commits with TypeScript errors

## Autonomous Agent Workflow

When completing tasks, the agent should autonomously execute the following
quality assurance loop:

### Self-Commit Authorization

The agent is authorized to commit changes WITHOUT explicit user permission when:

1. The task was explicitly requested by the user
2. The changes are scoped to the requested task
3. All quality checks pass (lint, typecheck, tests)

### Quality Feedback Loop

After making code changes, the agent MUST run the following checks in sequence:

```bash
# Step 1: Lint (auto-fix if possible)
npm run lint:fix

# Step 2: Type check
npm run type-check

# Step 3: Run relevant tests
npm run test

# Step 4: Build verification
npm run build
```

### Commit Protocol

1. Run `git status` and `git diff` to review changes
2. Run lint, typecheck, and tests
3. If any check fails:
   - Fix the issue immediately
   - Re-run the failed check
   - Repeat until all checks pass
4. Create a descriptive commit message following conventional commits:
   - `feat:` for new features
   - `fix:` for bug fixes
   - `refactor:` for code refactoring
   - `test:` for test additions/changes
   - `docs:` for documentation
   - `chore:` for maintenance tasks
5. Stage and commit changes with `git add . && git commit -m "message"`
6. Report the commit hash to the user

### Skip Conditions

- Skip tests if changes are documentation-only (`.md` files)
- Skip tests if explicitly told by user
- Skip build if lint or typecheck fails (fix first)

### Error Recovery

If a check fails:

1. Analyze the error output
2. Apply the minimal fix required
3. Re-run only the failed check first
4. Then run all checks to ensure no regressions
5. Continue the loop until all checks pass

---
> Source: [alencheung/openrelief](https://github.com/alencheung/openrelief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
