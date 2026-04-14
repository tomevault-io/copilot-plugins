## autonode

> AutoNode is a Next.js-based AI workflow automation platform that allows users to create and execute visual workflows with various node types (triggers, HTTP requests, AI actions).

# AutoNode - AI Workflow Automation Platform

AutoNode is a Next.js-based AI workflow automation platform that allows users to create and execute visual workflows with various node types (triggers, HTTP requests, AI actions).

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: Better Auth + Polar (payments)
- **API Layer**: tRPC
- **Background Jobs**: Inngest
- **State Management**: Jotai (atoms), React Query
- **UI**: Tailwind CSS v4, shadcn/ui, Radix UI
- **Workflow Editor**: React Flow (@xyflow/react)

---

## Build & Development Commands

This project uses [Task](https://taskfile.dev/) for running common commands. Make sure Task is installed, then use:

```bash
# Start database and dev server
task start

# Stop everything
task stop

# Start dev server only (Next.js + Inngest)
task dev

# Build for production
task build

# Run linter
task lint

# Database commands
task db:up      # Start Postgres container
task db:down    # Stop Postgres container
task db:push    # Push schema to database
task db:generate  # Generate migrations
task db:migrate  # Apply migrations
task db:studio  # Open Drizzle Studio
task db:reset   # Reset database (removes volumes)
```

Or use pnpm commands directly (if Task not available):

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Run linter
pnpm lint

# Run Inngest dev server (for background jobs)
pnpm inngest:dev

# Run all dev processes (Next.js + Inngest)
pnpm dev:all
```

### Running Tests

This project uses Vitest for testing. Run tests with:

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run a single test file
pnpm test -- path/to/test-file.test.ts

# Run tests matching a pattern
pnpm test -- --grep "workflow"
```

---

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Authentication pages (sign-in, sign-up)
│   ├── (dashboard)/       # Authenticated dashboard pages
│   ├── (marketing)/       # Public marketing pages
│   └── api/               # API routes (trpc, inngest, auth)
├── components/            # Shared UI components
│   ├── ui/               # shadcn/ui components
│   ├── react-flow/       # React Flow node components
│   └── dashboard/        # Dashboard-specific components
├── db/                   # Database layer
│   ├── schema/           # Drizzle schema definitions
│   ├── relations.ts      # Schema relations
│   └── index.ts          # DB client export
├── features/             # Feature-based modules
│   ├── credentials/       # API credentials management
│   ├── executions/       # Workflow execution logic
│   ├── triggers/         # Workflow triggers (manual, cron)
│   └── workflows/        # Workflow CRUD operations
├── hooks/                # Custom React hooks
├── inngest/              # Inngest functions (background jobs)
│   ├── functions/        # Individual Inngest functions
│   └── channels/        # Event channels
├── lib/                  # Utilities and config
│   ├── auth.ts           # Better Auth configuration
│   ├── polar.ts          # Polar payment client
│   └── utils.ts          # cn() utility (clsx + tailwind-merge)
├── services/             # External service integrations
└── trpc/                 # tRPC routers and procedures
    └── routers/          # API route handlers
```

---

## Code Style Guidelines

### Imports

- Use the `@/` alias for src-relative imports (configured in tsconfig.json)
- Order imports: external libs → internal imports → relative imports
- Use explicit imports over barrel exports where it improves readability

```typescript
// Good
import { useState } from "react";
import { useQuery } from "@tanstack/react-query";
import { db } from "@/db";
import { workflows } from "@/db/schema";
import { cn } from "@/lib/utils";

// Bad
import { db, workflows } from "@/db"; // Avoid barrel imports
```

### Naming Conventions

- **Files**: kebab-case for utilities, PascalCase for components and types
- **Functions/variables**: camelCase
- **Database columns**: snake_case
- **React components**: PascalCase
- **Types**: PascalCase with `Type` suffix (e.g., `WorkflowType`)

```typescript
// Component files
WorkflowEditor.tsx;
useWorkflows.ts;
node - selecter.tsx;

// Types
type WorkflowType = typeof workflows.$inferSelect;
type NodeEnumType = typeof nodeType.enumValues;
```

### TypeScript

- Always use explicit types for function parameters and return values
- Use type inference for local variables when type is obvious
- Use Zod for runtime validation (especially in tRPC inputs)

```typescript
// Good
export async function getWorkflow(
  id: number,
  userId: string,
): Promise<WorkflowType | undefined> {
  return db.query.workflows.findFirst({
    where: eq(workflows.id, id),
  });
}

// Zod schema for validation
const createWorkflowSchema = z.object({
  name: z.string().min(1),
  nodes: z.array(nodeSchema),
});
```

### React Patterns

- Use `"use client"` directive for client components
- Use `"use server"` for server actions
- Keep server/client boundary clear - fetch data in server components, mutate in client
- Use React Query (`useQuery`, `useMutation`) for server state
- Use Jotai atoms for client-side UI state

```typescript
// Server Action
"use server";

export async function createWorkflow(userId: string) {
  "use server";
  // Server-side logic
}

// Client Component
("use client");

import { useQuery } from "@tanstack/react-query";

export function WorkflowList() {
  const { data } = useQuery({
    queryKey: ["workflows"],
    queryFn: () => api.workflows.getMany.query(),
  });
}
```

### tRPC Procedures

Use the provided procedure helpers:

- `protectedProcedure` - Requires authentication
- `premiumProcedure` - Requires active subscription

```typescript
export const workflowsRouter = createTRPCRouter({
  getMany: protectedProcedure
    .input(
      z.object({
        page: z.number().default(1),
        search: z.string().default(""),
      }),
    )
    .query(async ({ ctx, input }) => {
      // Implementation
    }),

  create: premiumProcedure.mutation(async ({ ctx }) => {
    // Implementation - only for paid users
  }),
});
```

### Error Handling

- Throw `TRPCError` with appropriate code for API errors
- Use `NonRetriableError` from Inngest for permanent failures
- Use regular `Error` for retriable errors in Inngest functions

```typescript
// tRPC error
if (!workflow) {
  throw new TRPCError({
    code: "NOT_FOUND",
    message: "Workflow not found",
  });
}

// Inngest - non-retriable
throw new NonRetriableError("Workflow ID is required");

// Inngest - retriable (default)
throw new Error("Database temporarily unavailable");
```

### Database (Drizzle ORM)

- Use `pgTable` for table definitions
- Use `pgEnum` for enums
- Export inferred types with `$inferSelect` and `$inferInsert`
- Always use parameterized queries (Drizzle does this automatically)

```typescript
export const workflows = pgTable("workflows", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  userId: text("user_id")
    .notNull()
    .references(() => user.id, { onDelete: "cascade" }),
  createdAt: timestamp("created_at").notNull().defaultNow(),
});

export type WorkflowType = typeof workflows.$inferSelect;
```

### CSS & Styling

- Use Tailwind CSS utility classes
- Use the `cn()` utility (from `@/lib/utils`) for conditional classes
- Use `class-variance-authority` (cva) for component variants

```typescript
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva("base-classes", {
  variants: {
    variant: {
      default: "variant-classes",
      destructive: "destructive-classes",
    },
    size: {
      default: "size-default",
      sm: "size-sm",
    },
  },
});

export function Button({ className, variant, size }) {
  return (
    <button className={cn(buttonVariants({ variant, size }), className)} />
  );
}
```

### UI Components

- Use shadcn/ui components from `@/components/ui/`
- Use Radix UI primitives for new components
- Follow the component structure: component file + tests

---

## Planning Workflow

Before implementing features, always create a structured plan. See `docs/plan/init.md` for detailed instructions.

**Important**: When instructed to create a plan, only create the plan file. Do NOT start implementing it. Only execute a plan when explicitly instructed to "execute the plan" or "implement the plan from file".

### Adding Todo with Plan

When adding a feature to `@docs/todo.md`, always include the plan file path if one exists. This helps track which plan to follow:

```markdown
- [ ] Feature name ( description ) (Plan: docs/plan/features/feature-name/index.md)
```

### Quick Summary

1. **Create plan directory**: `docs/plan/<category>/<feature-name>/`
2. **Create plan file**: `docs/plan/<category>/<feature-name>/index.md`
3. **Include checklist**: Use `- [ ]` for pending, `- [x]` for completed
4. **Execute & track**: Mark tasks complete as you go
5. **Resume on error**: Reference last completed task in plan

All feature planning, modifications, errors, and related notes should stay in the same directory.

Example plan structure:

```markdown
# Feature Name

## Implementation Steps

### Step 1: Create database schema

- [ ] Add new table to schema
- [ ] Add migrations

### Step 2: Create API endpoint

- [ ] Add tRPC router
- [ ] Add validation

### Step 3: Create UI component

- [ ] Create component
- [ ] Add to page
```

---

## Common Patterns

### Creating a New Feature

1. Create database schema in `src/db/schema/`
2. Add tRPC router in `src/features/[feature]/server/routers.ts`
3. Create React components in `src/features/[feature]/components/`
4. Add hooks in `src/features/[feature]/hooks/`
5. Add Inngest functions if background processing needed

### Adding a New Node Type

1. Read `docs/plan/features/node.md` first for complete implementation guide
2. Add node type to `nodeType` enum in `src/db/schema/workflows.ts`
3. Create executor in `src/features/executors/lib/`
4. Register executor in `src/lib/node_executor_registery.ts`
5. Create node component and dialog in `src/features/executors/nodes/[node_name]_node/components/`
6. Register node component in `src/lib/node-components.tsx`
7. Add to sidebar in `src/components/react-flow/node-selecter.tsx`

### API Routes (tRPC)

All tRPC routers are auto-merged in `src/trpc/routers/_app.ts`. Add new routers there.

### Creating Forms

All forms taking user input must use:

- **react-hook-form** for form state management
- **zod** for schema validation
- **@hookform/resolvers/zod** for zod resolver

**Reference implementation:** `src/features/executions/components/http-node-dialog.tsx`

---

## Environment Variables

Required env vars (see `.env.example`):

- `DATABASE_URL` - PostgreSQL connection string
- `BETTER_AUTH_SECRET` - Auth encryption key
- `POLAR_ACCESS_KEY` - Polar payment API key
- `INNGEST_EVENT_KEY` - Inngest event key
- `INNGEST_SIGNING_KEY` - Inngest signing key

---

## Useful Commands

```bash
# Generate Drizzle migrations
pnpm drizzle-kit generate

# Push schema to database
pnpm drizzle-kit push

# Type-check the project
pnpm tsc --noEmit
```

---

## Plan Tracking Rules

When implementing features using plans:

### 1. Plan Updates

- After each todo is completed, update the plan file to mark it as `[x]`
- At the end of the plan, add a "Modified Files" section listing all changed/created files

### 2. Plan Completion

- When all implementation tasks are complete (not including testing), mark the plan as completed
- Move the plan directory to `docs/plan/<category>/archive/<feature-name>/`
- Add "Status: Completed" at the end of the plan

### 3. Plan Format

Each plan should end with:

```markdown
---

## Modified Files

### New Files Created
- `src/path/to/new-file.ts`

### Modified Files
- `src/path/to/existing-file.ts`

---

## Status: Completed
```

> **Important**: When executing a plan from `@docs/plan/...`, ALWAYS:
> 1. Update checkboxes `[ ]` to `[x]` as you complete each step
> 2. After all steps complete, update status to "Status: Completed"
> 3. Ensure "Modified Files" section lists all changes

> **Note:** For current project goals and todo items, see `docs/todo.md`
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ashutsoh1729) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
