## devl-dev

> Sean's scratch pad — a place to play with UI ideas built on `@orbit/ui` (coss-ui) primitives.

# CLAUDE.md

Sean's scratch pad — a place to play with UI ideas built on `@orbit/ui` (coss-ui) primitives.

## Layout

- `apps/www` — TanStack Start app (port 4000). The whole product.
- `packages/ui` — coss-ui components (55), Tailwind v4 + Base UI. Imported as `@orbit/ui/<component>`.

That's it — no backend, no auth, no database, no marketing site.

## Commands

```bash
npm run dev          # Start the scratchpad on :4000
npm run typecheck    # Type-check all workspaces
npm run build        # Build all workspaces
```

## How designs are organised

- `apps/www/src/lib/designs.ts` — single source of truth for categories and the designs inside each.
- `apps/www/src/components/wireframe-icons.tsx` — mini SVG wireframes used as folder icons on the index.
- `apps/www/src/pages/showcase/` — actual showcased designs (one component per design).
- `apps/www/src/pages/showcase/registry.tsx` — maps `(category, design-slug) → component`.

### Adding a new design

1. Add the showcase component in `apps/www/src/pages/showcase/<my-design>.tsx`.
2. Register it in `apps/www/src/pages/showcase/registry.tsx` under the right category.
3. Add a `{ slug, title, blurb }` entry to that category in `apps/www/src/lib/designs.ts`.

Design lives at `/c/<category>/<design-slug>`.

### Adding a new category

1. Add the wireframe SVG to `wireframe-icons.tsx`.
2. Add the category to `CATEGORIES` in `designs.ts` (and to the `CategorySlug` union).
3. Optionally add a `SHOWCASES[<slug>]` entry in `registry.tsx`.

---
> Source: [sean-brydon/devl.dev](https://github.com/sean-brydon/devl.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
