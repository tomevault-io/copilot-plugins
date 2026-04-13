## qc-admin

> CRM admin panel for Quant Club IIT KGP. Manages all content (blogs, whitepapers, events, team) and contact form submissions for the public `qc-frontend` site.

# qc-admin — Claude Code Instructions

## Project Purpose
CRM admin panel for Quant Club IIT KGP. Manages all content (blogs, whitepapers, events, team) and contact form submissions for the public `qc-frontend` site.

## Stack
- **Framework:** Next.js 16+ (App Router), React 19, TypeScript (strict, no `any`)
- **Styling:** Tailwind CSS v4, neobrutalism design system
- **Components:** Custom shadcn/ui-compatible components in `src/components/ui/`
- **Auth:** `iron-session` — `ADMIN_PASSWORD` env var, cookie-based sessions
- **Data:** JSON files in `data/` at project root — always use `src/lib/data-store.ts`
- **Package manager:** pnpm

## Design System (Neobrutalism)
Always use these conventions for interactive elements:
- `border-2 border-border` on cards, inputs, buttons
- `shadow-shadow` (`4px 4px 0px 0px black`) on elevated elements
- `bg-main` (green) for primary actions, `text-main-foreground` (black) for text on main
- `rounded-base` (5px) for all elements
- `font-heading` (700) for titles, `font-base` (500) for body text
- Hover: `hover:translate-x-boxShadowX hover:translate-y-boxShadowY hover:shadow-none`

## Auth Pattern
- `ADMIN_PASSWORD` env var compared against submitted password in `POST /api/auth`
- `SESSION_SECRET` env var (32+ chars) for iron-session encryption
- Middleware in `middleware.ts` protects all routes except `/login` and `/api/auth`
- Call `getSession()` from `@/lib/session` — never access cookies directly in components

## Data Layer
- All reads/writes go through `src/lib/data-store.ts` — never use `fs` directly in components
- JSON files are at `data/blogs.json`, `data/whitepapers.json`, `data/events.json`, `data/team.json`, `data/contacts.json`
- Contacts are append-only (no delete/edit)

## Server Actions Pattern
```ts
"use server"
import { revalidatePath } from "next/cache"
import { redirect } from "next/navigation"
import { addBlog } from "@/lib/data-store"

export async function createBlogAction(formData: FormData) {
  await addBlog({ ... })
  revalidatePath("/blogs")
  redirect("/blogs")
}
```

## Import Conventions
- Always use `@/` path alias (e.g., `@/lib/utils`, `@/components/ui/button`)
- No relative imports across feature boundaries

## Route Structure
- `/` → Dashboard (stat cards + chart + recent contacts)
- `/blogs` → CRUD for blog metadata
- `/whitepapers` → CRUD for whitepaper entries
- `/events` → CRUD for events
- `/team` → CRUD for team members
- `/contacts` → Read-only contact submissions viewer
- `/login` → Auth page (public)
- `/api/auth` → POST (login) / DELETE (logout)

## Custom Skills (`.claude/skills/`)
- `/new-blog` — add a blog entry to `data/blogs.json`
- `/new-event` — add an event entry to `data/events.json`
- `/add-team-member` — add a team member to `data/team.json`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quantclub-iitkgp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/quantclub-iitkgp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
