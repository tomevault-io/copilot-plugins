## nextjs-with-supabase

> This file provides context for Google Gemini when working with this Next.js + Supabase codebase.

# GEMINI.md - Google Gemini AI Context

This file provides context for Google Gemini when working with this Next.js + Supabase codebase.

---

## Project Summary

A Next.js 15 starter kit with Supabase authentication. Uses TypeScript, Tailwind CSS, and shadcn/ui components.

**Key Technologies:**
- Next.js 15 (App Router)
- React 19
- Supabase (Authentication)
- TypeScript 5
- Tailwind CSS 3.4
- shadcn/ui

---

## Project Structure

```
nextjs-with-supabase/
├── app/                    # Next.js App Router pages
│   ├── auth/               # Authentication routes
│   │   ├── confirm/        # Email confirmation
│   │   ├── login/          # Login page
│   │   ├── sign-up/        # Registration
│   │   ├── forgot-password/# Password reset
│   │   └── update-password/# New password
│   ├── protected/          # Auth-required routes
│   ├── layout.tsx          # Root layout
│   └── page.tsx            # Home page
├── components/             # React components
│   ├── ui/                 # shadcn/ui primitives
│   └── *-form.tsx          # Auth forms
├── lib/                    # Utilities
│   ├── supabase/           # Supabase clients
│   └── utils.ts            # Helper functions
└── middleware.ts           # Auth middleware
```

---

## Authentication System

### How It Works

1. **Supabase SSR Package** (`@supabase/ssr`)
   - Cookie-based session management
   - Works with Next.js server components

2. **Two Clients**
   - Browser: `lib/supabase/client.ts` (for forms)
   - Server: `lib/supabase/server.ts` (for pages)

3. **Middleware Protection**
   - `middleware.ts` checks auth on every request
   - Redirects to `/auth/login` if not authenticated
   - Allows `/` and `/auth/*` without auth

### Auth Flows

| Flow | Route | Form Component |
|------|-------|----------------|
| Login | `/auth/login` | `login-form.tsx` |
| Sign Up | `/auth/sign-up` | `sign-up-form.tsx` |
| Forgot Password | `/auth/forgot-password` | `forgot-password-form.tsx` |
| Reset Password | `/auth/update-password` | `update-password-form.tsx` |
| Email Confirm | `/auth/confirm` | Route handler |

---

## Key Code Patterns

### Server Component (Protected Page)

```typescript
// app/protected/page.tsx
import { redirect } from "next/navigation";
import { createClient } from "@/lib/supabase/server";

export default async function ProtectedPage() {
  const supabase = await createClient();
  const { data, error } = await supabase.auth.getClaims();

  if (error || !data?.claims) {
    redirect("/auth/login");
  }

  return <div>Protected content</div>;
}
```

### Client Component (Form)

```typescript
// components/login-form.tsx
"use client";

import { useState } from "react";
import { createClient } from "@/lib/supabase/client";

export function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [isLoading, setIsLoading] = useState(false);

  const handleLogin = async (e: React.FormEvent) => {
    e.preventDefault();
    const supabase = createClient();
    setIsLoading(true);

    const { error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });

    if (!error) {
      router.push("/protected");
    }

    setIsLoading(false);
  };

  return <form onSubmit={handleLogin}>...</form>;
}
```

### Styling with Tailwind

```typescript
// Using cn() utility for conditional classes
import { cn } from "@/lib/utils";

<div className={cn(
  "flex flex-col gap-6",
  isActive && "bg-accent",
  className
)}>
```

---

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_OR_ANON_KEY=your-anon-key
```

Both are `NEXT_PUBLIC_*` (exposed to browser, safe for anon key).

---

## Common Tasks

### Add New Protected Page

1. Create `app/new-page/page.tsx`
2. Add auth check:
   ```typescript
   const supabase = await createClient();
   const { data, error } = await supabase.auth.getClaims();
   if (error || !data?.claims) redirect("/auth/login");
   ```

### Add New UI Component

```bash
npx shadcn@latest add button
```

### Create Form Component

1. Create `components/my-form.tsx`
2. Add `"use client"` at top
3. Use `createClient()` from `@/lib/supabase/client`
4. Handle loading and error states

---

## Important Files

| File | Purpose |
|------|---------|
| `middleware.ts` | Route protection |
| `lib/supabase/client.ts` | Browser Supabase client |
| `lib/supabase/server.ts` | Server Supabase client |
| `lib/supabase/middleware.ts` | Session refresh logic |
| `lib/utils.ts` | Utility functions |
| `app/globals.css` | Theme variables |
| `tailwind.config.ts` | Tailwind configuration |

---

## Scripts

```bash
npm run dev    # Development server
npm run build  # Production build
npm run lint   # ESLint
```

---

## Current Status

**Completed:**
- Full email/password auth
- Theme switching (dark/light)
- Protected routes
- UI component library

**Needed:**
- Form validation (Zod)
- User profiles
- OAuth providers
- Testing setup

---

## Working Guidelines

1. **Server vs Client Components**
   - Default to Server Components
   - Use Client Components only for interactivity
   - Mark with `"use client"` directive

2. **Type Safety**
   - TypeScript strict mode enabled
   - Type all function parameters
   - Use `unknown` over `any`

3. **Styling**
   - Use Tailwind utilities
   - Use `cn()` for conditional classes
   - Follow shadcn/ui patterns

4. **Error Handling**
   - Always try/catch async operations
   - Display user-friendly error messages
   - Log errors appropriately

---

## Related Documentation

- [README.md](README.md) - Project overview
- [CLAUDE.md](CLAUDE.md) - AI assistant context
- [AGENTS.md](AGENTS.md) - Multi-agent workflows
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) - System architecture
- [docs/ROADMAP.md](docs/ROADMAP.md) - Development roadmap

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Krosebrook)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Krosebrook)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
