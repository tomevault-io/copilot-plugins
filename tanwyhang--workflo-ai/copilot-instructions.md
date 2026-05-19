## workflo-ai

> WhatsApp-first AI customer service platform built with Next.js 16, Supabase, and AI integration.

# Workflo App - AI Agent Instructions

## Project Overview
WhatsApp-first AI customer service platform built with Next.js 16, Supabase, and AI integration.

## Tech Stack
- **Framework**: Next.js 16.1 (App Router)
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS 4 + shadcn/ui (new-york style)
- **State**: Zustand (stores) + React Context (auth/workspace)
- **Database**: Supabase (PostgreSQL + Auth + Storage)
- **AI**: OpenAI (GPT-4o-mini, embeddings)
- **Validation**: Zod v4
- **Toasts**: sonner

## Build/Lint/Test Commands
```bash
npm run dev      # Start dev server
npm run build    # Production build (always verify before committing)
npm run lint     # ESLint (flat config, eslint-config-next)
```
No test runner is configured (no jest/vitest). No `*.test.*` or `*.spec.*` files exist.

## Naming Conventions
- **Files**: `kebab-case.tsx` (e.g., `chat-list.tsx`), pages use `page.tsx`
- **Components**: PascalCase (e.g., `KnowledgeSection`)
- **Exports**: Named exports preferred (`export function Foo()`)
- **Props**: `ComponentNameProps`
- **Stores**: `*-store.ts` or `store.ts` in `lib/stores/`
- **Types**: `kebab-case.ts` (e.g., `types.ts`)

## Import Organization
Group imports in this order:
1. React/Next.js
2. External packages (lucide-react, sonner, zustand, etc.)
3. Internal aliases (`@/components/*`, `@/lib/*`)
4. Relative imports (`./`, `../`)

Use barrel exports where they exist: `import { Button, Card } from "@/components/ui"`. Do NOT add barrel files unless the directory already has one.

## Directory Structure
```
app/
├── api/              # API routes
├── workspace/[id]/   # Authenticated workspace pages
├── onboarding/       # Onboarding flow
└── page.tsx          # Landing / redirect

components/
├── ui/               # shadcn/ui primitives (DO NOT EDIT)
├── providers/        # React Context providers
├── sidebar.tsx       # Main navigation sidebar
├── chat/             # Chat feature components
├── ai/               # AI config components
├── dashboard/        # Dashboard widgets
└── test-ai/          # AI testing UI

lib/
├── api/              # API utilities (auth, validation, response)
├── rbac/             # Auth context, guards, permissions
├── supabase/         # Database clients + generated types
├── ai/               # RAG, chatbot, embeddings, vector store
├── whatsapp/         # WhatsApp client + messaging
├── stores/           # Zustand stores (use this, not root lib/)
├── hooks/            # Custom React hooks
└── utils.ts          # Shared utilities (cn, etc.)
```

## Supabase Clients
Three clients — use the right one:
- **Browser** (`lib/supabase/client.ts`): Client components, RLS applies
- **Server** (`lib/supabase/server.ts`): Server components/actions, RLS applies with user session
- **Admin** (`lib/supabase/admin.ts`): API routes, bypasses RLS — use sparingly

Always check auth in API routes: `const { data: { user } } = await supabase.auth.getUser()`

## State Management
- **Global state**: Zustand stores in `lib/stores/`
- **Auth/Workspace**: React Context (`AuthProvider` in root layout, `WorkspaceProvider` in workspace layout)
- **Local state**: `useState` in components
- **Persisted state**: Zustand `persist` middleware (see `workspace-store.ts`, `onboarding-store.ts`)

## API Route Patterns
```typescript
import { NextResponse } from "next/server";
import { createClient } from "@/lib/supabase/server";
import { z } from "zod";

const schema = z.object({ name: z.string().min(1) });

export async function POST(request: Request) {
  const supabase = await createClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });

  const body = schema.parse(await request.json());
  // ... logic
  return NextResponse.json({ data: result });
}
```

## Component Patterns

Page components use `"use client"` and `useAuth()`. Feature components use `"use client"` with `ComponentNameProps` interfaces. Use RBAC guards for permission-based rendering: `<Can>`, `<RoleGuard>`, `<FeatureGuard>`, `<AuthGuard>`.

## Common Patterns

### Adding a new page
1. Create `app/workspace/[id]/feature/page.tsx`
2. Add route to sidebar in `components/sidebar.tsx`
3. Add RBAC permission if needed in `lib/rbac/permissions.ts`

### Adding a new API endpoint
1. Create `app/api/feature/route.ts`
2. Use shared auth/validation from `lib/api/`
3. Return `NextResponse.json({ data })` or `NextResponse.json({ error }, { status: 400 })`

### Adding a new store
1. Create in `lib/stores/feature-store.ts`
2. Export as `useFeatureStore`

### Adding shadcn components
```bash
npx shadcn@latest add component-name
```
This auto-installs to `components/ui/`. Do NOT edit those files manually.

## Do NOT
- Do NOT edit files in `components/ui/` directly (use `npx shadcn@latest add`)
- Do NOT use `as any` unless absolutely necessary (add proper types instead)
- Do NOT create stores in root `lib/` (use `lib/stores/`)
- Do NOT mix external/internal imports (keep them grouped)
- Do NOT use default exports for components (use named exports)
- Do NOT add barrel `index.ts` files unless the directory already has one

## Database Types
- Types are in `lib/supabase/database.types.ts` (auto-generated)
- Regenerate with: `supabase gen types typescript`
- For missing tables, use `as any` temporarily with a `// TODO:` comment

## Integration Rule (CRITICAL)
No feature is "Done" until it is verified in the Render Tree. After creating a new file:
1. Find its intended parent component
2. Verify the import statement is added
3. Verify the component is rendered/called
4. Never leave "Ghost Code" (code that is written but unreachable)

---
> Source: [Tanwyhang/Workflo-AI](https://github.com/Tanwyhang/Workflo-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
