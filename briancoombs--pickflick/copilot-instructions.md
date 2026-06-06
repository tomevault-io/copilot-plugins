## auth

> Follow these rules when working on auth.

### Auth Rules

Follow these rules when working on auth.

It uses Supabase Auth for authentication.

#### General Rules

- Import the server client with `import { createClient } from "@/lib/supabase/server"` or `import { getAuthenticatedUser } from "@/lib/supabase/server"` in server components and actions
- Import the browser client with `import { createClient } from "@/lib/supabase/client"` in client components
- Use the `useUser()` hook from `@/hooks/use-user` in client components to get the current user
- Use `getAuthenticatedUser()` in server actions for a quick auth check
- Auth state is managed through cookies — no React context provider needed
- Middleware at `/lib/supabase/middleware.ts` handles session refresh and route protection
- Login/signup server actions are in `/app/(auth)/login/actions.ts`

---
> Source: [BrianCoombs/pickflick](https://github.com/BrianCoombs/pickflick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
