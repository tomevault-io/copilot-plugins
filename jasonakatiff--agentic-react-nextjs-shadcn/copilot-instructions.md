## agentic-react-nextjs-shadcn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Agent-ready SaaS dashboard — greenfield Next.js project optimized for AI agent testability via `agent-browser`. The core idea: write React that keeps DOM as source of truth (like HTMX) while retaining rich interactivity.

## Stack

- Next.js (App Router) + TypeScript (strict) + Tailwind CSS
- shadcn/ui (built on Radix primitives)
- Neon Postgres + Drizzle ORM
- pnpm, Vitest, agent-browser (e2e), Vercel

## Commands

```bash
pnpm dev              # dev server
pnpm build            # production build
pnpm lint             # eslint (jsx-a11y strict mode)
pnpm test             # vitest unit tests
pnpm test <file>      # single test file
pnpm db:push          # drizzle schema push
pnpm db:generate      # drizzle migrations
pnpm db:studio        # drizzle studio
```

## Setup

```bash
pnpm install
cp ~/.config/templates/.env.local .
cp ~/.config/templates/.env.example .
```

## Architecture: "Server-First React"

The guiding principle is agent-testability — an `agent-browser snapshot` should see real, interactive content immediately.

### Server Components by default
Every page/layout is RSC. Only wrap the interactive bit (dropdown, datepicker, modal) in `'use client'` as a small "island." Parent pages stay RSC so agents see full structure.

### Server Actions for mutations
Use `<form action={serverAction}>` with named inputs and `<button type="submit">`, not `onClick` + `fetch('/api/...')`. Agents see real `<form>` elements.

### State in URL, not useState
Use `searchParams` in server components for tab state, filters, pagination. `<Link href="?tab=overview">` instead of `useState('overview')`. Agents can navigate directly.

### Semantic HTML contract
- Clickable → `<button>` or `<a>`, never `<div onClick>`
- Lists → `<ul>`/`<li>`, not nested divs
- Every interactive element gets `aria-label` or visible text
- Every form input gets `<label>` or `aria-label`
- Icon-only buttons require `aria-label`
- Enforced via `eslint-plugin-jsx-a11y` (strict mode)

### Agent-aware loading states
```tsx
<Suspense fallback={<div role="status" aria-label="Loading leads">...</div>}>
  <LeadsTable />
</Suspense>
```

### Client islands pattern
Complex widgets (rich dropdowns, datepickers, modals) wrapped with agent-friendly attributes:
```tsx
<div role="combobox" aria-label={label} aria-expanded={open}>
  {/* rich UI */}
</div>
```

### What stays client-side
Optimistic UI, drag-and-drop, real-time collab, animations, client-side filtering, offline — these are `'use client'` islands wrapped in semantic, aria-labeled containers.

## CI

GitHub Actions runs agent-browser e2e tests against deployed previews.

---
> Source: [jasonakatiff/agentic-react-nextjs-shadcn](https://github.com/jasonakatiff/agentic-react-nextjs-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
