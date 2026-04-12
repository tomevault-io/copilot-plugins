## quickstarter-4-monorepo-nextjs-supabase

> This is a monorepo containing a Next.js application with Supabase and Prisma integration, managed with pnpm workspaces.

# Claude Development Guide

This is a monorepo containing a Next.js application with Supabase and Prisma integration, managed with pnpm workspaces.

## Project Structure

```
├── packages/
│   └── web/                    # Next.js app
│       ├── app/                # App Router (Next.js 15+)
│       ├── lib/
│       │   ├── supabase/       # Supabase client configurations
│       │   ├── prisma.ts       # Prisma client singleton
│       │   └── generated/prisma/ # Generated Prisma client
│       └── prisma/
│           └── schema.prisma   # Database schema
├── package.json               # Root package.json with workspace scripts
├── pnpm-workspace.yaml       # Workspace configuration
└── .gitignore                # Git ignore rules
```

## Development Commands

### Start Development Server

```bash
pnpm dev                    # Start web package dev server
pnpm --filter web dev       # Alternative syntax
```

### Build and Deploy

```bash
pnpm build                  # Build web package
pnpm start                  # Start production server
pnpm lint                   # Run linting
```

### Database Operations

```bash
pnpm --filter web exec prisma db pull          # Pull schema from Supabase
pnpm --filter web exec prisma generate         # Generate Prisma client
pnpm --filter web exec prisma migrate dev      # Run migrations
```

### Testing Commands

```bash
pnpm test                   # Run all tests
pnpm test:watch            # Run tests in watch mode
pnpm test:ui               # Open Vitest web UI
pnpm test:coverage         # Run tests with coverage
```

## Import Aliasing

**IMPORTANT**: Always use import aliasing (`@/*`) when importing files within the Next.js application. This project is configured with TypeScript path mapping for cleaner imports.

### ✅ Correct Usage:

```typescript
import Component from '@/components/MyComponent';
import { prisma } from '@/lib/prisma';
import { createClient } from '@/lib/supabase/client';
```

### ❌ Avoid Relative Imports:

```typescript
import { createClient } from '../../../lib/supabase/client';
import { prisma } from '../../lib/prisma';
import Component from './components/MyComponent';
```

## Database Integration

### Supabase Client

- **Browser**: `@/lib/supabase/client` - For client-side operations
- **Server**: `@/lib/supabase/server` - For server-side operations with cookies

### Prisma Client

- Import from: `@/lib/prisma`
- Pre-configured singleton to prevent multiple instances
- Schema auto-generated from Supabase database

### Example Usage:

```typescript
// API Route example
import { prisma } from '@/lib/prisma';
import { createClient } from '@/lib/supabase/client';
import { NextResponse } from 'next/server';

export async function GET() {
  // Use Supabase for real-time features
  const supabase = createClient();
  const { data } = await supabase.from('users').select('*');

  // Use Prisma for complex queries
  const userCount = await prisma.user.count();

  return NextResponse.json({ data, userCount });
}
```

## Environment Variables

Environment variables are configured in:

- `.env.local` - Next.js environment variables
- `.env` - Prisma environment variables

Required variables:

- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `DATABASE_URL` (for Prisma)
- `DIRECT_URL` (for Prisma migrations)

## Testing Integration

This project uses **Vitest** for testing with jsdom environment and comprehensive coverage reporting.

### Test File Naming and Organization

**CRITICAL CONVENTIONS:**

- **ALWAYS** use `filename.test.ts` naming convention - NEVER use `__tests__` directories
- **ALWAYS** colocate test files with the source files they test
- Use `@/*` import aliasing in test files

### ✅ Correct Test File Structure:

```
lib/
├── utils.ts
├── utils.test.ts          # Colocated with utils.ts
├── supabase/
│   ├── client.ts
│   └── client.test.ts     # Colocated with client.ts
components/
├── Button.tsx
└── Button.test.tsx        # Colocated with Button.tsx
```

### ❌ Avoid **tests** Directories:

```
lib/
├── utils.ts
└── __tests__/             # DON'T DO THIS
    └── utils.test.ts      # Should be colocated instead
```

### Test Environment

- **Framework**: Vitest with jsdom environment
- **Coverage**: Configured for text, JSON, and HTML reports
- **Global**: `describe`, `it`, `expect`, `vi` available globally
- **Aliases**: `@/*` paths work in tests

Test API endpoint available at: `/api/test-prisma`

- Verifies Supabase connection
- Verifies Prisma connection
- Returns connection status

## Key Features

- **Monorepo**: pnpm workspaces for efficient dependency management
- **Next.js 15**: Latest with App Router and Turbopack
- **Supabase**: Real-time database and authentication
- **Prisma**: Type-safe database client with auto-generated types
- **TypeScript**: Full type safety across the stack
- **Import Aliasing**: Clean imports with `@/*` paths

## Notes for Claude

1. Always use `@/` import aliasing - never relative imports
2. Use pnpm workspace commands: `pnpm --filter web <command>`
3. Prisma client is a singleton - import from `@/lib/prisma`
4. Environment variables are split between `.env.local` and `.env`
5. Database schema is pulled from Supabase, not manually created
6. Test connectivity with `/api/test-prisma` endpoint
7. **CRITICAL**: Use `filename.test.ts` naming - NEVER `__tests__/` directories
8. **CRITICAL**: Always colocate test files with source files they test

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martink-rsa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/martink-rsa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
