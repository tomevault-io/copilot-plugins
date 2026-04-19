## kb-tutor

> An MCQ-based learning support web app with three roles: student, teacher, and admin.

# kb-tutor

An MCQ-based learning support web app with three roles: student, teacher, and admin.

## Tech Stack

- **Framework**: Next.js 16 (App Router), React 19, TypeScript
- **Styling**: Tailwind CSS v4
- **DB / Auth**: Supabase (`@supabase/ssr`)
- **Testing**: Vitest + jsdom
- **AI**: Google Gemini (`@google/generative-ai`) — question generation
- **Math**: KaTeX / react-katex
- **Other**: Framer Motion, Recharts, lucide-react

## Commands

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run lint         # ESLint
npm test             # Vitest (run once)
npm run test:watch   # Vitest (watch mode)
npm run test:coverage
```

## Directory Structure

```
src/
  app/               # Next.js App Router pages + API Routes
    api/             # Route Handlers (admin/, assignments/, auth/, teacher/)
    content/         # Admin content management pages
    teacher-dashboard/
    assignments/
    login/
  components/        # React components
    modes/           # Practice mode UI (ReviewMode, etc.)
  lib/
    auth/            # Role resolution utilities (role.ts, types.ts)
    supabase/        # Supabase client factories (client.ts, server.ts, env.ts)
    storage.ts       # Hybrid localStorage + Supabase storage
    question-storage.ts
    gemini.ts        # Gemini API calls
  hooks/             # Custom React hooks
  types/             # Shared type definitions (question.ts)
supabase/
  migrations/        # SQL migration files
```

## Auth & Roles

Three roles: `student` / `teacher` / `admin`.

| Path                                        | Required role            |
| ------------------------------------------- | ------------------------ |
| `/login`                                    | Public                   |
| `/teacher-dashboard`, `/assignments/manage` | teacher or admin         |
| `/content`                                  | admin                    |
| All others                                  | Authenticated (any role) |

- Role resolution priority: `profiles.role` → `user_metadata.role` → `app_metadata.role`
- Use the correct Supabase client for each context:
  - **Browser**: `getSupabaseBrowserClient()` (`src/lib/supabase/client.ts`)
  - **Server Component / Route Handler**: `createSupabaseServerClient()` (`src/lib/supabase/server.ts`)
  - **Middleware**: use `createServerClient` directly

<important if="writing code that uses Supabase">
- Always use `createSupabaseServerClient()` in Server Components.
- Access env vars via the getter functions in `src/lib/supabase/env.ts` — do not reference `process.env` directly.
- `SUPABASE_SERVICE_ROLE_KEY` is server-side only. Never expose it to the client.
- Add new migrations to `supabase/migrations/`.
</important>

<important if="writing API Route Handlers">
- Do not rely solely on middleware for auth checks. Always re-verify with `supabase.auth.getUser()` inside the Route Handler.
- For role-protected endpoints, fetch the role from the `profiles` table and validate it in the handler.
</important>

## UI Language

All user-facing text must be written in English. This includes:

- Labels, headings, and button text
- Error messages and validation feedback
- Empty states and loading messages
- Tooltips, placeholders, and helper text
- Any instructional or descriptive copy

<important if="adding or editing any UI text">
Write all user-facing strings in English. Do not use Japanese or any other language for text that appears in the UI.
</important>

## Coding Conventions

- **Types**: Never use `any`. Use `unknown` with type guards when the type is uncertain.
- **Server / Client split**: Add `"use client"` only when necessary. Prefer Server Components for data fetching.
- **Storage**: Always guard `localStorage` access with `typeof window !== "undefined"` for SSR safety.
- **Math**: Use `react-katex` for rendering math expressions. Do not use `dangerouslySetInnerHTML`.

## Testing

- Place test files alongside the source file as `*.test.ts`.
- Prioritize testing pure utility functions in `src/lib/`.
- Use `vi.mock()` to mock Supabase and external APIs.

## Environment Variables

| Variable                        | Purpose                                        |
| ------------------------------- | ---------------------------------------------- |
| `NEXT_PUBLIC_SUPABASE_URL`      | Supabase project URL                           |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anonymous key                         |
| `SUPABASE_SERVICE_ROLE_KEY`     | Server-side admin key (never expose to client) |
| `NEXT_PUBLIC_GEMINI_API_KEY`    | Gemini API key                                 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miinakoyama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
