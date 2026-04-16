## goat-sports

> You are working on GOAT Sports, a next-generation fantasy sports platform built with:

# GOAT Sports - Cursor AI Rules

## Project Context
You are working on GOAT Sports, a next-generation fantasy sports platform built with:
- **Framework**: Next.js 15 (App Router) with TypeScript
- **Database**: PostgreSQL via Drizzle ORM
- **UI**: shadcn/ui components (Radix UI primitives) with Tailwind CSS
- **Package Manager**: Bun (preferred) or npm
- **State Management**: React Query (TanStack Query) for server state

## Beads Issue Tracking

This project uses **beads** (`bd`) for persistent issue tracking across sessions.

### Core Commands
```bash
bd ready                 # Find available work (no blockers)
bd show <id>             # View issue details with dependencies
bd list --status=open    # All open issues
bd create --title="..." --type=task|bug|feature --priority=2
bd update <id> --status=in_progress  # Claim work
bd close <id>            # Complete work
bd close <id1> <id2>     # Close multiple issues efficiently
bd sync                  # Sync beads changes with git
bd dep add <issue> <depends-on>  # Add dependency
bd blocked               # Show all blocked issues
```

### When to Use Beads
- Multi-session work that needs persistence
- Complex tasks with dependencies or blockers
- Strategic work that survives conversation compaction
- Discovered work during implementation
- Parallel work streams

### Priority Levels
Use numeric priorities: `0-4` or `P0-P4`
- **P0/0**: Critical (production down, data loss)
- **P1/1**: High (blocks key features)
- **P2/2**: Medium (normal work, default)
- **P3/3**: Low (nice to have)
- **P4/4**: Backlog (future consideration)

### Session Completion - MANDATORY

**Before ending ANY session, complete ALL steps:**

```bash
# 1. Create issues for remaining work
bd create --title="..." --type=task --priority=2

# 2. Run quality gates (if code changed)
bun run type-check && bun run lint && bun run test

# 3. Update beads status
bd close <id1> <id2>  # Close completed work
bd update <id> --notes="Current status..."  # Update in-progress

# 4. Commit code changes
git add <files>
git commit -m "Description

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

# 5. Push to remote (MANDATORY - work is NOT done until this succeeds)
git pull --rebase
bd sync
git push
git status  # MUST show "up to date with origin"
```

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - stranded local work is lost work
- If push fails, resolve conflicts and retry until success
- Create issues for ALL remaining work before ending session

## Code Style & Conventions

### TypeScript
- Use strict TypeScript with full type inference
- Leverage Drizzle's type inference from schema - never manually type database results
- Use `type` for type aliases, `interface` for object shapes that may be extended
- Prefer explicit return types for functions that return complex types
- Use `as const` for literal types when needed

### File Organization
- **API Routes**: `app/api/[resource]/route.ts` - use Next.js 15 App Router conventions
- **Components**: `components/[feature]/ComponentName.tsx` - feature-based organization
- **UI Components**: `components/ui/` - shadcn/ui components (don't modify these directly)
- **Database**: `lib/db/schema.ts` - single source of truth for all database schemas
- **Utils**: `lib/utils/` - shared utility functions
- **Types**: Inline with components/API routes, or in `lib/types/` if shared
- **Documentation**: All documentation files (except README.md) go in `docs/` folder

### Naming Conventions
- **Components**: PascalCase (e.g., `PlayerCard.tsx`)
- **Functions**: camelCase (e.g., `calculateFantasyPoints`)
- **Constants**: UPPER_SNAKE_CASE for true constants, camelCase for exported configs
- **Database tables**: plural, lowercase (e.g., `players`, `teams`)
- **Database columns**: camelCase in schema, snake_case in DB (Drizzle handles conversion)

### Database Patterns
- **Always use Drizzle ORM** - never raw SQL unless absolutely necessary
- **Batch queries** - avoid N+1 queries, use `inArray()` for batch fetching
- **Type inference** - use `typeof schema.players.$inferSelect` for types
- **Relations** - define in `schema.ts` using Drizzle relations API
- **Migrations** - always generate via `bun run db:generate` after schema changes

### API Route Patterns
```typescript
// Standard API route structure
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';
import { players } from '@/lib/db/schema';
import { eq, and, or } from 'drizzle-orm';

export async function GET(request: NextRequest) {
  try {
    if (!db) {
      return NextResponse.json(
        { error: 'Database not configured' },
        { status: 503 }
      );
    }
    // Implementation
    return NextResponse.json({ data });
  } catch (error) {
    console.error('Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Component Patterns
- **Server Components by default** - use `'use client'` only when needed (interactivity, hooks, browser APIs)
- **Type props explicitly** - use TypeScript interfaces for component props
- **Use shadcn/ui components** - import from `@/components/ui/`
- **Tailwind classes** - use utility classes, avoid inline styles
- **Accessibility** - all interactive elements must be keyboard accessible

### Performance Guidelines
- **Avoid N+1 queries** - batch database operations
- **Use React Query** for server state caching and refetching
- **Optimize imports** - use named imports, avoid barrel exports in hot paths
- **Code splitting** - use dynamic imports for heavy components
- **Image optimization** - use Next.js Image component

### Error Handling
- **API Routes**: Return appropriate HTTP status codes (400, 404, 500, 503)
- **Components**: Use error boundaries for component-level errors
- **Database**: Always handle null/undefined cases from database queries
- **User-facing**: Show friendly error messages, log technical details to console

### Testing Approach
- **Type checking**: Run `bun run type-check` before committing (enforces strict checks)
- **Linting**: Run `bun run lint` to catch issues
- **Manual testing**: Test in browser, especially for UI components
- **Performance testing**: Check for N+1 queries, verify batch operations work correctly

## Common Patterns

### Fantasy Points Calculation
```typescript
import { calculateFantasyPoints } from '@/lib/utils/fantasy';
// Use this utility for all fantasy point calculations
```

### Database Queries
```typescript
// Good: Batch query
const teamIds = results.map(r => r.teamId).filter(Boolean);
const teams = await db.select().from(teams).where(inArray(teams.id, teamIds));

// Bad: N+1 query
for (const result of results) {
  const team = await db.select().from(teams).where(eq(teams.id, result.teamId));
}
```

### Component Props
```typescript
interface PlayerCardProps {
  player: Player;
  showStats?: boolean;
  onSelect?: (player: Player) => void;
}

export function PlayerCard({ player, showStats = false, onSelect }: PlayerCardProps) {
  // Implementation
}
```

## When Making Changes

1. **Schema Changes**: Update `lib/db/schema.ts`, then run `bun run db:generate` to create migration
2. **New API Route**: Create `app/api/[resource]/route.ts` following the pattern above
3. **New Component**: Create in `components/[feature]/` with proper TypeScript types
4. **New Utility**: Add to `lib/utils/` with JSDoc comments explaining usage
5. **Dependencies**: Use `bun add [package]` for dependencies, `bun add -d [package]` for devDeps
6. **Documentation**: Place all documentation files (except README.md) in `docs/` folder

## Avoid Over-Engineering

Keep solutions simple and focused:
- Only make changes that are directly requested or clearly necessary
- Don't add features, refactoring, or "improvements" beyond what was asked
- A bug fix doesn't need surrounding code cleaned up
- A simple feature doesn't need extra configurability
- Don't add docstrings, comments, or type annotations to unchanged code
- Only add comments where logic isn't self-evident
- Don't add error handling for scenarios that can't happen
- Trust internal code and framework guarantees
- Only validate at system boundaries (user input, external APIs)
- Don't create helpers/utilities for one-time operations
- Don't design for hypothetical future requirements
- Three similar lines of code > premature abstraction
- If something is unused, delete it completely (no backwards-compatibility hacks)

## Important Notes

- **Never commit `.env.local`** - it's in `.gitignore`
- **Database schema is the source of truth** - types are inferred from schema
- **Use path aliases** - `@/` maps to project root
- **Bun is preferred** - but npm works too
- **Performance matters** - optimize queries, avoid unnecessary re-renders, batch database operations
- **Type safety first** - leverage TypeScript and Drizzle's type inference
- **Documentation location** - All docs (except README.md) belong in `docs/` folder
- **TypeScript dev builds** - `noUnusedLocals` and `noUnusedParameters` disabled for faster dev, but still checked via `bun run type-check`
- **Node.js version** - Using Node.js 24.12.0 (LTS) or Bun 1.0+

## AI Assistant Guidelines

When generating code:
- Always use proper TypeScript types
- Follow existing patterns in the codebase
- Add JSDoc comments for complex functions
- Consider performance implications
- Handle edge cases (null, undefined, empty arrays)
- Use Drizzle ORM patterns, not raw SQL
- Import from correct paths using `@/` alias
- Use shadcn/ui components when appropriate
- Follow Next.js 15 App Router conventions
- **Documentation**: Place all documentation files (except README.md) in `docs/` folder
- **Performance**: Always batch database queries to avoid N+1 problems
- **Package Manager**: Use Bun commands (`bun add`, `bun run`, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/isellar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
