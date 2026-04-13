## lts-csp

> LTS Tax is an AI-powered, automated tax compliance and return filing platform. It processes Guernsey Economic Substance Register (ESR) returns and other tax jurisdictions using browser automation, AI document extraction, and multi-modal processing.

# LTS Tax - Claude Code Context

## Project Overview

LTS Tax is an AI-powered, automated tax compliance and return filing platform. It processes Guernsey Economic Substance Register (ESR) returns and other tax jurisdictions using browser automation, AI document extraction, and multi-modal processing.

**Core functionality:**
- Automated tax return processing (reduces 45min manual entry to 2min)
- AI-powered document extraction from PDFs and Excel files
- Secure browser automation with human-in-the-loop capabilities
- Multi-jurisdiction support (Guernsey primary, Jersey/Isle of Man/Luxembourg planned)

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 15.2, React 19, TypeScript 5.8, TailwindCSS 4.0, Shadcn/ui |
| API | tRPC 11 (type-safe RPC) |
| Database | PostgreSQL + Drizzle ORM 0.41 |
| State | Zustand 5 (client), React Query 5 (server), Redis (real-time) |
| AI | Vercel AI SDK, Anthropic Claude, Gemini AI |
| Automation | Browser Use Cloud API |
| Infrastructure | Vercel (web), AWS ECS Fargate (background jobs), Vercel Blob (storage) |
| Auth | Supabase Auth + SSR |

## Monorepo Structure

```
LTS/
├── apps/web/              # Main Next.js application
│   ├── src/app/           # App Router pages
│   ├── src/server/        # tRPC routers and API
│   ├── src/components/    # React components
│   └── src/lib/           # Utilities (analytics, feature-flags, ecs)
├── packages/
│   ├── database/          # Drizzle schema and queries
│   └── redis/             # Pub/sub utilities
├── containers/
│   ├── task/              # Browser automation task runner (ECS)
│   └── tax-sync/          # Tax portal scraper (ECS)
└── scripts/               # Utility scripts
```

## Common Commands

```bash
# Development
pnpm dev                   # Start all apps in dev mode
pnpm build                 # Build all packages
pnpm lint                  # Run ESLint

# Database
pnpm db:generate           # Generate Drizzle migrations
pnpm db:migrate            # Run migrations
pnpm db:push               # Push schema changes
pnpm db:studio             # Open Drizzle Studio

# Docker (containers)
pnpm docker:build          # Build container images
```

## Key Files

| File | Purpose |
|------|---------|
| `apps/web/src/server/api/root.ts` | tRPC router aggregation |
| `apps/web/src/server/api/trpc.ts` | tRPC setup & middleware |
| `apps/web/src/lib/ecs.ts` | AWS ECS task launcher |
| `apps/web/src/lib/analytics.ts` | Event tracking system |
| `apps/web/src/lib/feature-flags.ts` | Vercel Edge Config integration |
| `packages/database/src/schema.ts` | All database table definitions (784 lines) |
| `containers/task/src/index.ts` | Main browser automation logic |

## Database Schema (key tables)

- `accounts` - User profiles (extends Supabase auth)
- `organisations` - Multi-tenant organizations
- `org_members` - Team members with roles (owner/admin/member)
- `tax_returns` - Tax returns to process
- `substance_forms` - Guernsey ESR form data (40+ fields, 11 sections)
- `tasks` - Automation tasks (tax_return, validation, submission, etc.)
- `jobs` - Task execution attempts with retry tracking
- `jurisdictions` - Tax jurisdictions and portal credentials

## Code Conventions

### TypeScript
- Strict mode enabled
- Zod for runtime validation
- Discriminated unions for state machines

### API
- tRPC for internal type-safe API
- Next.js Route Handlers for external endpoints
- Context-based dependency injection

### Components
- React Server Components by default
- `"use client"` only at component boundary
- Shadcn/ui component library

### State Management
- Server state: tRPC + React Query
- Client state: Zustand stores
- Real-time: Redis pub/sub

## Important Patterns

### tRPC Router Structure
```typescript
// apps/web/src/server/api/routers/example.ts
export const exampleRouter = createTRPCRouter({
  getAll: protectedProcedure.query(async ({ ctx }) => {
    // ctx.db, ctx.session available
  }),
  create: protectedProcedure
    .input(z.object({ name: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // ...
    }),
});
```

### Database Queries
```typescript
import { db } from "@lts/database";
import { taxReturns } from "@lts/database/schema";

const returns = await db.select().from(taxReturns).where(eq(taxReturns.orgId, orgId));
```

### ECS Task Launch
```typescript
import { launchEcsTask } from "@/lib/ecs";

await launchEcsTask({
  taskType: "tax_return",
  taskId: task.id,
  jobId: job.id,
});
```

## Environment Variables

Key variables needed (see `.env.example`):
- `DATABASE_URL` - PostgreSQL connection
- `REDIS_URL` - Redis connection
- `AWS_*` - AWS credentials and ECS config
- `ANTHROPIC_API_KEY` - Claude AI
- `BROWSER_USE_API_KEY` - Browser automation
- `SUPABASE_*` - Authentication

## Deployment

- **Web**: Vercel (auto-deploy on push)
- **Tasks**: AWS ECS Fargate (EU-West-2/London)
- **Storage**: Vercel Blob, PostgreSQL
- **CDN**: Vercel Edge Network

## Testing Approach

When making changes:
1. Run `pnpm lint` for type errors
2. Run `pnpm build` to verify compilation
3. Test locally with `pnpm dev`
4. Database changes require migration generation

## Security Notes

- Portal credentials encrypted in `jurisdiction_settings`
- Supabase handles auth tokens
- Job isolation with unique credentials per execution
- GDPR-compliant UK infrastructure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flawstick)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/flawstick)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
