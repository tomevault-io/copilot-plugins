## ecomup-canvas

> Single-user infinite canvas web app for a Performance Marketing Specialist managing Facebook Ads across 7 markets (UK, US, AU, CA, NZ, DE, VN). Consolidates all scattered work resources into one visual workspace.

# EcomUp Canvas - Project Context

## What is this?
Single-user infinite canvas web app for a Performance Marketing Specialist managing Facebook Ads across 7 markets (UK, US, AU, CA, NZ, DE, VN). Consolidates all scattered work resources into one visual workspace.

## Tech Stack
- **Frontend:** Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS 3
- **State:** Zustand
- **DnD:** @dnd-kit/core + @dnd-kit/sortable
- **Canvas:** Custom (CSS transforms + HTML divs + SVG connectors) — NOT React Flow, NOT Konva, NOT tldraw
- **Backend:** Supabase (PostgreSQL + Auth + Storage)
- **Hosting:** Vercel
- **Repo:** GitHub (thnglee)

## Supabase
- Project ref: `kqrlbbonqarnydhbyruo`
- Region: Northeast Asia (Seoul)
- Org ID: `iaribncdbtqlaotebufa`

## Key Architecture Decisions
- Custom canvas using CSS `transform: translate() scale()` for pan/zoom — GPU-accelerated with `will-change: transform`
- Components are absolutely positioned HTML divs (native form/table/markdown support)
- Connectors rendered in SVG overlay layer
- Zones render below components (z-index starts at -1000)
- JSONB columns for flexible component data
- Row Level Security for data isolation
- Auto-save with debounce, optimistic updates

## Design Rules
- Dark theme throughout (#0a0a0f background, #1a1a2e cards)
- 20px snap grid
- 40px max header height (world clock bar)
- Viewport culling for performance (render only visible + 200px buffer)

## Development Phases
- **Phase 1 (Sprint 1-2):** Foundation — Canvas, pan/zoom, grid, auth, Supabase schema, auto-save
- **Phase 2 (Sprint 3-4):** Components — Sidebar, drag-drop, 5 component types (LinkBox, StickyNote, DataTable, ProcessBlock, ImageBadge)
- **Phase 3 (Sprint 5):** Connectors & Zones — SVG connectors, zone grouping, group-move
- **Phase 4 (Sprint 6):** Polish — Undo/redo, keyboard shortcuts, minimap, performance
- **Phase 5 (Sprint 7):** Deploy & Data — Production deploy, data population, testing

## Current Status
- **Phase:** Phase 5 in progress (Sprint 7: Deploy & Data)
- **Completed:** All Phase 1-4 features + auth hardening (login rate limiting 5-attempt lockout, logout button, redirect flows), export to JSON (toolbar download), Vercel deployment (https://ecomup-one.vercel.app), GitHub repo (thnglee/ecomup-canvas), env vars configured
- **Next:** Data population (7.6), end-to-end testing (7.7), bug fixes (7.9)

## Files to Read for Full Context
- `PRD.md` — Full product requirements with acceptance criteria
- `DEV_PLAN.md` — Detailed sprint breakdown with task-level estimates

## Conventions
- Always read PRD.md and DEV_PLAN.md before implementing a sprint task
- Update "Current Status" section in this file when completing a sprint
- Commit after each meaningful task completion
- Keep components in `src/components/canvas/` for canvas-related, `src/components/ui/` for shared UI
- Store types in `src/types/`
- Zustand stores in `src/store/`
- Supabase client in `src/lib/supabase.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thnglee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thnglee)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
