## finance-scoop

> This document provides context and guidelines for AI assistants working with the Finance Scoop codebase.

# CLAUDE.md - AI Assistant Guide

This document provides context and guidelines for AI assistants working with the Finance Scoop codebase.

---

## Project Overview

**Finance Scoop** is a Next.js 16 application that monitors social media (Reddit, and soon X/Twitter) for finance discussions, detects Lona promotion opportunities using AI (Grok), and generates smart reply drafts.

**Repository**: https://github.com/iamtxena/finance-scoop
**Tech Stack**: Next.js 16, React 19, TypeScript, TanStack React Query, Zustand, Clerk, Supabase, xAI (Grok)

---

## Architecture Principles

### 1. Modern React Patterns (NO useEffect!)
- **Use TanStack React Query** for all data fetching
- **Use Zustand** for form state and complex client state
- **Server Components** for initial data fetching
- **Client Components** for interactivity and real-time updates

```typescript
// ❌ AVOID - Old pattern with useEffect
useEffect(() => {
  fetch('/api/alerts').then(res => setData(res));
}, []);

// ✅ CORRECT - Use React Query
const { data: alerts } = useQuery({
  queryKey: ['alerts'],
  queryFn: async () => {
    const { data } = await axios.get('/api/alerts');
    return data.alerts;
  },
});
```

### 2. File Structure Conventions
```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Route groups (don't affect URL)
│   ├── (dashboard)/       # Protected routes
│   └── api/               # API route handlers
├── components/
│   ├── ui/                # shadcn/ui base components (don't edit directly)
│   └── features/          # Feature-specific components
├── hooks/                 # React Query hooks (use-*.ts pattern)
├── stores/                # Zustand stores (*-store.ts pattern)
├── lib/                   # Utility libraries
│   ├── supabase/         # Database clients
│   ├── reddit/           # Reddit API wrapper
│   ├── ai/               # AI agents and prompts
│   ├── redis/            # Cache helpers
│   └── notifications/    # Email/Slack
└── types/                 # TypeScript type definitions
```

### 3. Import Order (MUST FOLLOW)
```typescript
// 1. External imports
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. Internal components
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

// 3. Hooks
import { useAlerts } from '@/hooks/use-alerts';

// 4. Lib/utils
import { createClient } from '@/lib/supabase/client';

// 5. Types
import type { Alert } from '@/hooks/use-alerts';
```

**Critical**: User has specified in their global CLAUDE.md:
> DO NOT CREATE IMPORTS IN THE MIDDLE OF THE CODE. everything at the top of the file

---

## Key Patterns

### API Routes (Next.js 15+)

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  // 1. Auth check
  const { userId } = await auth();
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // 2. Await params (Next.js 15+)
  const { id } = await params;

  // 3. Business logic
  // ...

  return NextResponse.json({ data });
}
```

### React Query Hooks

```typescript
// hooks/use-alerts.ts
export function useAlerts() {
  return useQuery({
    queryKey: ['alerts'],
    queryFn: async () => {
      const { data } = await axios.get('/api/alerts');
      return data.alerts as Alert[];
    },
  });
}

export function useCreateAlert() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (input: CreateAlertInput) => {
      const { data } = await axios.post('/api/alerts', input);
      return data.alert as Alert;
    },
    onSuccess: () => {
      // Invalidate cache for automatic refetch
      queryClient.invalidateQueries({ queryKey: ['alerts'] });
    },
  });
}
```

### Zustand Stores

```typescript
// stores/alert-form-store.ts
import { create } from 'zustand';

interface AlertFormState {
  keywords: string[];
  addKeyword: (keyword: string) => void;
  removeKeyword: (index: number) => void;
  reset: () => void;
}

export const useAlertFormStore = create<AlertFormState>((set) => ({
  keywords: [],
  addKeyword: (keyword) =>
    set((state) => ({ keywords: [...state.keywords, keyword] })),
  removeKeyword: (index) =>
    set((state) => ({ keywords: state.keywords.filter((_, i) => i !== index) })),
  reset: () => set({ keywords: [] }),
}));
```

### Supabase Client Usage

```typescript
// Server-side (API routes)
import { createClient } from '@/lib/supabase/server';

const supabase = await createClient();
const { data, error } = await supabase
  .from('alerts')
  .select('*')
  .eq('user_id', userId);

// Client-side (React components)
import { createClient } from '@/lib/supabase/client';

const supabase = createClient();
// Same query syntax
```

### AI Agent Usage

```typescript
import { analyzeSentiment, draftReply } from '@/lib/ai/agents';

// All AI calls are automatically traced with LangSmith
const sentiment = await analyzeSentiment(postContent);
const draft = await draftReply(postContent);
```

---

## Common Tasks

### 1. Adding a New API Route

```bash
# Create directory
mkdir -p src/app/api/new-feature

# Create route.ts
touch src/app/api/new-feature/route.ts
```

```typescript
// src/app/api/new-feature/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@clerk/nextjs/server';

export async function GET(request: NextRequest) {
  const { userId } = await auth();
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Your logic here

  return NextResponse.json({ data: 'success' });
}
```

### 2. Adding a New React Query Hook

```typescript
// src/hooks/use-new-feature.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import axios from 'axios';

export function useNewFeature() {
  return useQuery({
    queryKey: ['new-feature'],
    queryFn: async () => {
      const { data } = await axios.get('/api/new-feature');
      return data;
    },
  });
}

export function useCreateNewFeature() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (input: any) => {
      const { data } = await axios.post('/api/new-feature', input);
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['new-feature'] });
    },
  });
}
```

### 3. Adding a New Page

```typescript
// src/app/(dashboard)/new-page/page.tsx
'use client';

import { useNewFeature } from '@/hooks/use-new-feature';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

export default function NewPage() {
  const { data, isLoading } = useNewFeature();

  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">New Page</h1>
      <Card>
        <CardHeader>
          <CardTitle>Data</CardTitle>
        </CardHeader>
        <CardContent>
          {isLoading ? 'Loading...' : JSON.stringify(data)}
        </CardContent>
      </Card>
    </div>
  );
}
```

### 4. Adding a New Database Table

```sql
-- supabase/migrations/002_new_table.sql
CREATE TABLE IF NOT EXISTS new_table (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id TEXT NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_new_table_user_id ON new_table(user_id);

ALTER TABLE new_table ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own rows"
  ON new_table FOR SELECT
  USING (auth.jwt() ->> 'sub' = user_id);

CREATE POLICY "Users can manage their own rows"
  ON new_table FOR ALL
  USING (auth.jwt() ->> 'sub' = user_id);
```

Then update `src/lib/supabase/types.ts` with the new table definition.

### 5. Adding a New AI Agent

```typescript
// src/lib/ai/agents.ts
import { xai } from '@ai-sdk/xai';
import { traceable } from 'langsmith/traceable';
import { generateText } from './client';

export const newAgent = traceable(
  async (input: string): Promise<string> => {
    const { text } = await generateText({
      model: xai('grok-4-fast-reasoning'),
      prompt: `Your prompt here: ${input}`,
      temperature: 0.3,
    });

    return text.trim();
  },
  {
    name: 'new-agent-name',
    run_type: 'llm',
    metadata: {
      model: 'grok-4-fast-reasoning',
      task: 'description',
    },
  }
);
```

---

## Important Code Patterns

### Error Handling in API Routes

```typescript
export async function POST(request: NextRequest) {
  try {
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const body = await request.json();

    // Validation
    if (!body.requiredField) {
      return NextResponse.json(
        { error: 'requiredField is required' },
        { status: 400 }
      );
    }

    // Business logic
    const result = await someOperation(body);

    return NextResponse.json({ data: result });
  } catch (error) {
    console.error('Error in POST /api/route:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Loading States in UI

```typescript
'use client';

export default function Page() {
  const { data, isLoading, error } = useQuery({ ... });

  if (isLoading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error: {error.message}</div>;
  }

  if (!data || data.length === 0) {
    return <div>No data found</div>;
  }

  return <div>{/* Render data */}</div>;
}
```

### Optimistic Updates

```typescript
const updateAlert = useUpdateAlert();

const handleToggle = async (id: string, currentActive: boolean) => {
  await updateAlert.mutateAsync({
    id,
    input: { active: !currentActive },
  });
};
```

React Query automatically:
- Shows loading state during mutation
- Refetches data on success
- Reverts on error (if configured)

---

## Redis Caching Pattern

```typescript
// lib/reddit/client.ts
import { cacheGet, cacheSet } from '../redis/client';

export async function getUserPosts(username: string) {
  // Try cache first
  const cacheKey = `reddit:user:${username}:posts`;
  const cached = await cacheGet<RedditPost[]>(cacheKey);
  if (cached) return cached;

  // Fetch from Reddit API
  const posts = await fetchFromReddit(username);

  // Cache for 5 minutes
  await cacheSet(cacheKey, posts, 300);

  return posts;
}
```

---

## Debugging

### LangSmith Traces
All AI operations are automatically traced. View at:
https://smith.langchain.com/

### Supabase Logs
Database queries and RLS policies:
Supabase Dashboard > Logs

### Vercel Logs
API routes and cron jobs:
Vercel Dashboard > Project > Logs

### Local Development
```bash
# Check environment variables
printenv | grep -i clerk
printenv | grep -i supabase

# Test API route
curl http://localhost:3000/api/alerts \
  -H "Authorization: Bearer $(clerk-token)"
```

---

## Cron Job Testing

```bash
# Test locally (with valid CRON_SECRET)
curl -X GET http://localhost:3000/api/cron/check-reddit \
  -H "Authorization: Bearer your_cron_secret"

# Test on Vercel
curl -X GET https://your-app.vercel.app/api/cron/check-reddit \
  -H "Authorization: Bearer your_cron_secret"
```

---

## Code Style

### TypeScript
- Use explicit types for function parameters and returns
- Avoid `any` - use `unknown` or proper types
- Use `interface` for objects, `type` for unions

### React Components
- Use functional components only
- Prefer arrow functions for component definitions
- Use `'use client'` directive when needed

### Naming Conventions
- Components: PascalCase (`AlertCard.tsx`)
- Hooks: camelCase with `use` prefix (`use-alerts.ts`)
- Stores: camelCase with `-store` suffix (`alert-form-store.ts`)
- API routes: kebab-case (`check-reddit/route.ts`)
- Types: PascalCase (`Alert`, `RedditPost`)

---

## Testing Checklist

When making changes:

1. **Authentication**: Test both logged-in and logged-out states
2. **API Routes**: Verify auth checks, error handling, validation
3. **React Query**: Check loading, error, and success states
4. **Mutations**: Verify cache invalidation works
5. **Database**: Check RLS policies allow/deny correctly
6. **UI**: Test responsive design (mobile, tablet, desktop)
7. **AI Calls**: Verify LangSmith traces appear
8. **Cron Jobs**: Test manually before relying on schedule

---

## Common Pitfalls

### ❌ Don't: Import in middle of file
```typescript
import { useState } from 'react';

function Component() {
  import { Button } from '@/components/ui/button'; // ❌ WRONG
}
```

### ✅ Do: All imports at top
```typescript
import { useState } from 'react';
import { Button } from '@/components/ui/button'; // ✅ CORRECT

function Component() {
  // ...
}
```

### ❌ Don't: Use useEffect for data fetching
```typescript
useEffect(() => {
  fetch('/api/data').then(res => setData(res)); // ❌ WRONG
}, []);
```

### ✅ Do: Use React Query
```typescript
const { data } = useQuery({ // ✅ CORRECT
  queryKey: ['data'],
  queryFn: fetchData,
});
```

### ❌ Don't: Forget to await params in Next.js 15+
```typescript
export async function GET(request, { params }) {
  const { id } = params; // ❌ WRONG - params is a Promise
}
```

### ✅ Do: Await params
```typescript
export async function GET(request, { params }: { params: Promise<{ id: string }> }) {
  const { id } = await params; // ✅ CORRECT
}
```

---

## Environment Variables Reference

**Required for Development**:
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
- `CLERK_SECRET_KEY`
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`
- `XAI_API_KEY`
- `REDDIT_APP_ID`
- `REDDIT_APP_SECRET`
- `REDDIT_USERNAME`
- `REDDIT_PASSWORD`
- `UPSTASH_REDIS_REST_URL`
- `UPSTASH_REDIS_REST_TOKEN`

**Optional**:
- `LANGCHAIN_API_KEY` (for LangSmith)
- `RESEND_API_KEY` (for email)
- `SLACK_WEBHOOK_URL` (for Slack)
- `CRON_SECRET` (for securing cron endpoint)

---

## Useful Commands

```bash
# Development
pnpm dev                    # Start dev server with Turbopack
pnpm build                  # Build for production
pnpm start                  # Start production server
pnpm lint                   # Run ESLint

# Database
# Run migrations in Supabase dashboard SQL editor

# Git
git status                  # Check changes
git add -A                  # Stage all changes
git commit -m "message"     # Commit
git push origin main        # Push to GitHub

# Dependencies
pnpm add package-name       # Add dependency
pnpm add -D package-name    # Add dev dependency
pnpm remove package-name    # Remove dependency
```

---

## Resources

- **Setup Guide**: [SETUP.md](./SETUP.md)
- **Project README**: [README.md](./README.md)
- **Next.js Docs**: https://nextjs.org/docs
- **React Query Docs**: https://tanstack.com/query/latest
- **Clerk Docs**: https://clerk.com/docs
- **Supabase Docs**: https://supabase.com/docs
- **AI SDK Docs**: https://ai-sdk.dev/docs
- **LangSmith Docs**: https://docs.langchain.com/langsmith

---

## Contributing

When adding new features:

1. Follow existing patterns (see examples above)
2. Add React Query hooks for data fetching
3. Use Zustand for complex form state
4. Include error handling in API routes
5. Add proper TypeScript types
6. Test authentication and authorization
7. Update this CLAUDE.md if adding new patterns

---

## Support

For issues or questions:
- Create GitHub issue
- Check SETUP.md for configuration help
- Review LangSmith traces for AI debugging
- Check Vercel/Supabase logs for runtime errors

---

**Last Updated**: November 2025
**Maintainer**: @iamtxena

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iamtxena)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iamtxena)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
