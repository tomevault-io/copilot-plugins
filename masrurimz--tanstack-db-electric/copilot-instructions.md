## tanstack-db-electric

> This is a TanStack Start project with oRPC for mutations and Electric sync for reads, running on Start's server functions so it's easily deployable to many hosting platforms.

# TanStack Start + DB + Electric Starter

This is a TanStack Start project with oRPC for mutations and Electric sync for reads, running on Start's server functions so it's easily deployable to many hosting platforms.

**Core Pattern**: Electric SQL for reads, oRPC for writes, TanStack DB for optimistic updates.

All reads from the Postgres database are done via the Electric sync engine. All mutations (create, update, delete) are done via oRPC with full end-to-end type safety.

We sync normalized data from tables into TanStack DB collections in the client & then write client-side queries for displaying data in components.

## General Usage

You MUST read https://electric-sql.com/AGENTS.md for general information on developing wth Electric and TanStack DB.

## Starter template specifics

You CAN choose to read the `README.md` for this project if useful. Much of it is summarized below.

### Pre-reqs

Docker, Caddy (with root cert installed using `caddy trust`), and Bun. Versions in `.tool-versions`.

### Initial setup

```sh
bun install    # install deps
bun backend:up # run backend services using docker compose
bun migrate    # apply migrations
bun dev        # run dev server
```

The dev server runs over HTTPS via a CaddyPlugin in the vite config. This supports HTTP/2 which is essential to avoid slow shapes for Electric.

### Linting and formatting

Human devs have IDEs that autoformat code on every file save. After you edit files, you must do the equivalent by running `bun lint`.

This command will also report linter errors that were not automatically fixable. Use your judgement as to which of the linter violations should be fixed.

### Build/Test Commands

- `bun dev` - Start development server with Docker services
- `bun build` - Build for production
- `bun test` - Run all Vitest tests
- `vitest run <test-file>` - Run single test file
- `bun start` - Start production server

### Architecture

- **Frontend**: TanStack Start (SSR framework for React and other frameworks) with file-based routing in `src/routes/`
- **Database**: PostgreSQL with Drizzle ORM, schema in `src/db/schema.ts`
- **Electric**: Real-time sync service on port 30000
- **Services**: Docker Compose setup (Postgres on 54321, Electric on 30000)
- **Styling**: Tailwind CSS v4
- **Authentication**: better-auth
- **API**: oRPC for mutations with full e2e type safety, Electric shapes for real-time reads
- **Validation**: Arktype with drizzle-arktype for schema generation

### API Routing

- **oRPC** (`/api/rpc/*`) - All mutations (create, update, delete) with full type safety
- **better-auth** (`/api/auth/*`) - Authentication endpoints
- **Electric shapes** (`/api/projects`, `/api/todos`, `/api/users`) - Real-time sync endpoints for reads

### Code Style

- **TypeScript**: Strict mode, ES2022 target, bundler module resolution
- **Imports**: Use `@/*` path aliases for `src/` directory imports
- **Components**: React 19 with JSX transform, functional components preferred
- **Server DB**: Drizzle ORM with PostgreSQL dialect, schema-first approach
- **Client DB**: TanStack DB with Electric Sync Collections
- **Routing**: File-based with TanStack Router, use `Link` component for navigation
- **Testing**: Vitest with @testing-library/react for component tests
- **file names** should always use kebab-case

### oRPC Integration

- oRPC routers are defined in `src/lib/orpc/routers/` directory
- Base procedures in `src/lib/orpc/index.ts` with `publicProcedure` and `protectedProcedure`
- Context creation in `src/lib/orpc/context.ts`
- Client is configured in `src/lib/orpc/client.ts`
- API handler in `src/routes/api/rpc/$.ts`
- Collection hooks use oRPC client for mutations in `src/lib/collections.ts`
- Transaction IDs are generated using `pg_current_xact_id()::xid::text` for Electric sync compatibility

### Data Flow Architecture

#### Reading Data (Electric SQL → TanStack DB)

```tsx
// 1. Preload in route loader
export const Route = createFileRoute('/todos/')({
  loader: async () => {
    await Promise.all([
      todosCollection.preload(),
      projectsCollection.preload(), // Include if used by child components
    ])
  },
})

// 2. Query with useLiveQuery (ALWAYS destructure data)
const { data: todos } = useLiveQuery(
  (q) => q.from({ todosCollection }).where(...),
  [dependencies] // Include reactive dependencies
)
```

#### Writing Data (TanStack DB → oRPC)

```tsx
// Use collection operations for optimistic updates
todosCollection.insert({ ... })  // NOT orpc.todos.create()
// Similar to Immer
todosCollection.update(id, (draft) => { ... })
todosCollection.delete(id)
```

#### Collection Definition Pattern

```tsx
// src/lib/collections.ts
export const todosCollection = createCollection(
  electricCollectionOptions({
    id: 'todos',
    shapeOptions: { url: '/api/todos', ... },
    schema: selectTodosSchema,
    getKey: (item) => item.id,

    // oRPC handlers (CRUD only, return { txid })
    onInsert: async ({ transaction }) => {
      const result = await orpc.todos.create({ ... })
      return { txid: result.txid }
    },
    onUpdate: async ({ transaction }) => { ... },
    onDelete: async ({ transaction }) => { ... },
  })
)
```

#### oRPC Router Pattern

```tsx
// src/lib/orpc/routers/todos.ts
import { type } from "arktype"
import { ORPCError } from "@orpc/server"
import { protectedProcedure, generateTxId } from "@/lib/orpc"

export const todosRouter = {
  create: protectedProcedure
    .input(createTodoSchema)
    .handler(async ({ context, input }) => {
      const result = await context.db.transaction(async (tx) => {
        const txid = await generateTxId(tx)
        const [newItem] = await tx.insert(todosTable).values(input).returning()
        return { item: newItem, txid }
      })
      return result
    }),
}
```

### Critical Rules

1. **NEVER use oRPC for data reads** - Only Electric SQL + useLiveQuery
2. **NEVER call oRPC directly from components** - Use collection operations
3. **NEVER use TanStack Query** - This uses TanStack DB (different library)
4. **ALWAYS preload collections** in route loaders
5. **ALWAYS use snake_case** for database fields throughout the app
6. **ONLY basic CRUD in oRPC** - No special mutations unless using `createOptimisticAction`

### Naming Conventions

- **Database**: snake_case (e.g., `user_id`, `created_at`)
- **Files**: kebab-case (e.g., `todo-card.tsx`)
- **Routes**: Use `_` prefix for pathless layouts (e.g., `_authenticated.tsx`)

### Schema Management

```tsx
// src/db/schema.ts (centralized, never redefine)
import { createSelectSchema, createInsertSchema, createUpdateSchema } from "drizzle-arktype"

export const selectTodoSchema = createSelectSchema(todosTable)
export const createTodoSchema = createInsertSchema(todosTable).omit(`created_at`)
export const updateTodoSchema = createUpdateSchema(todosTable)

// Type inference uses typeof Schema.infer
export type Todo = typeof selectTodoSchema.infer
```

### Component Patterns

- **Forms**: Use optimistic updates, no loading states needed
- **Links**: Use TanStack Router's `Link` component
- **Auth**: Access via `authClient.useSession()`

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masrurimz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/masrurimz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
