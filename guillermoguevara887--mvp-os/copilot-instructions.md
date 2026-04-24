## mvp-os

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev        # Start development server at localhost:3000
pnpm build      # Production build
pnpm lint       # ESLint
```

## Environment Variables

Requires `.env.local` with:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `OPENAI_API_KEY`
- `OPENAI_MODEL` (optional, defaults to `gpt-4.1-mini`)

## Architecture

**MVPOS** is a SaaS tool for generating AI-powered MVP execution plans. Users create projects, and the app automatically generates a structured plan and sprints via OpenAI.

### Auth & Routing

- Auth is handled by Supabase via `middleware.ts`, which guards `/dashboard` (redirects to `/login` if unauthenticated) and redirects logged-in users away from `/login`.
- Two Supabase clients exist: `lib/supabase/client.ts` (browser) and `lib/supabase/server.ts` (server/API routes). Use the correct one per context.

### AI Pipeline

Two separate AI flows:

1. **MVP Plan generation** (client-side, triggered on project creation):
   - `hooks/use-projects.ts` → `lib/ai/generate-plan.ts` → `lib/ai/build-prompt.ts` + `lib/ai/generate-mvp.ts`
   - Result stored in `project_ai_requests` table.

2. **Sprint generation** (server-side, via API route):
   - `POST /api/projects/[projectId]/generate-sprints` → `lib/server/generateProjectSprints.ts`
   - Reads the latest entry from `project_ai_requests`, builds a sprint prompt, calls OpenAI, then inserts into `project_sprint_generations`, `project_sprints`, and `project_sprint_tasks`.

### Supabase Tables

- `projects` — user projects
- `project_ai_requests` — raw AI prompt+response for MVP plan
- `project_sprint_generations` — sprint generation metadata
- `project_sprints` — individual sprints with status/order
- `project_sprint_tasks` — tasks per sprint

### UI Structure

- `app/dashboard/page.tsx` — main shell: sidebar + workspace, client component
- `components/app-sidebar.tsx` — project list navigation
- `components/project-workspace.tsx` — tabbed workspace (Definicion, Sprints, Docs, API, Prompts)
- `components/tabs/` — one file per workspace tab
- `components/ui/` — shadcn/ui primitives (do not edit directly)

### Styling

Tailwind CSS v4 with shadcn/ui. Use `cn()` from `lib/utils.ts` for conditional class merging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guillermoguevara887) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
