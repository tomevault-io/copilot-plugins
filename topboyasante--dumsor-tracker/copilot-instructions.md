## dumsor-tracker

> Guidance for AI agents working in this repo.

# CLAUDE.md

Guidance for AI agents working in this repo.

## What this is

Odumsor — a community power-tracker for Ghana. React 19 + TypeScript + Vite + Tailwind v4 + shadcn/ui, served as an SPA. Backend is separate; this app talks to it via the URL in `VITE_API_BASE_URL`.

## Toolchain

- Use **Bun**, not npm or yarn. The lockfile is `bun.lock`.
- Common commands:
  - `bun run dev` — start the dev server
  - `bun run typecheck` — type-check without emitting
  - `bun run lint` — ESLint
  - `bun run build` — type-check and production build
  - `bun run format` — Prettier
- Before claiming work is done, run `bun run typecheck` and `bun run lint`.

## Project layout

```
src/
  routes/         route components (lazy-loaded in router.tsx)
  components/
    ui/           shadcn primitives — generated, edit with care
    zones/        zone list, map, bottom nav
    schedules/    ECG schedule pages
    about/        about page
    admin/        admin-only views
  lib/
    api/          TanStack Query hooks + fetch helpers (single source of API truth)
    utils.ts      `cn()` and other small utilities
  hooks/          shared React hooks
  router.tsx      react-router-dom v7 routes (lazy imports)
  main.tsx        app entry
```

Path alias: `@/` → `src/`.

## Conventions to follow

- **Toasts: use `sileo`, not `sonner`.** Both are in `package.json` but only `sileo` is wired to the mounted `<Toaster>` (see `src/components/app-toaster.tsx`). Calls to `sonner.toast(...)` will silently no-op.
- **Routing:** `react-router-dom` v7. Routes are lazy-loaded in `src/router.tsx` — add new routes there.
- **Data fetching:** Use the hooks in `@/lib/api` (TanStack Query). Don't call `fetch` directly from components; add a hook there instead.
- **Styling:** Tailwind v4. Compose conditional classes with `cn()` from `@/lib/utils`. Prefer existing tokens (`bg-background`, `text-muted-foreground`, `text-brand`, etc.) over raw colors.
- **shadcn components:** `bunx shadcn@latest add <name>`. They land in `src/components/ui` — treat that directory as generated code, but you can hand-edit when needed.
- **Icons:** `lucide-react`.
- **Env vars:** Vite-style. Only `VITE_*` vars reach the client. Default `VITE_API_BASE_URL` is `http://localhost:8000`.

## Things that look like patterns but aren't

- `sonner` in `package.json` ≠ "we use sonner." See above.
- `src/components/admin/` exists but admin features aren't gated by an env-injected key on the client right now. Don't reintroduce `VITE_ADMIN_API_KEY` — it was removed intentionally.

## When making UI changes

- Test in the browser before reporting done. Type-checks pass ≠ feature works.
- Keep components consistent with the design language already in `src/components/zones` and `src/components/about` — `font-serif` for headings, muted helper text, generous spacing, rounded cards (`rounded-xl border bg-muted/40`).
- Mobile-first. The app is laid out for phone widths (`max-w-md`) with `BottomNav`. Don't break that.

## What to avoid

- Don't add backwards-compat shims, deprecation comments, or "removed because" markers. Just delete.
- Don't add comments that restate what the code does. Only add a comment when the *why* is non-obvious.
- Don't introduce a second toast library, second router, or second data-fetching layer. Use what's already wired.
- Don't commit unless the user asks.

---
> Source: [topboyasante/dumsor-tracker](https://github.com/topboyasante/dumsor-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
