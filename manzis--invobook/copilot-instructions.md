## vercel-design

> Vercel design system — use Design/vercel preview HTMLs and tokens for all UI work


# Invobook Vercel UI (mandatory for visual work)

## Canonical design sources (read before editing UI)

1. `Design/vercel/preview.html` — **light mode reference** (exact CSS for components)
2. `Design/vercel/preview-dark.html` — dark mode reference
3. `Design/vercel/DESIGN.md` — rules, typography scale, do's/don'ts
4. `Design/vercel/tokens.css` — CSS variables (`--ds-*`)
5. `Design/vercel/COMPONENT-MAP.md` — preview class → `ds-*` utilities

Default theme: **light** (match `preview.html`).

## Hard rules

- Use **shadow-as-border** (`var(--ds-shadow-ring)`), not `border border-gray-200` on cards/panels.
- Text primary: `#171717` (`--ds-black`), never pure `#000` for UI chrome.
- Body/secondary text: `#4d4d4d` (`--ds-gray-600`).
- Canvas/background: `#ffffff` (`--ds-white`), subtle tint `#fafafa` (`--ds-gray-50`).
- Font: **Geist** + **Geist Mono** with `font-feature-settings: "liga" 1`.
- Weights: **400** body, **500** UI/links/buttons, **600** headings only (no 700 except micro-badges).
- Display letter-spacing: **-2.4px** at 48px, **-1.28px** at 32px, **-0.96px** at 24px.
- Primary CTA: `ds-btn-dark` (#171717 bg). Secondary: `ds-btn-ghost`.
- Prefer `ds-*` classes from `src/styles/globals.css` over inventing new hex values.
- Workflow accents (Develop Blue, Preview Pink, Ship Red) only for pipeline/workflow UI — not general decoration.
- Do **not** use legacy Invobook blue (`bg-blue-600`, `from-blue-500`) when building or converting pages.

## New pages

1. Identify matching sections in `preview.html` (nav, hero, cards, forms, tables).
2. Compose layout with `ds-section`, `ds-card`, `ds-btn-*`, `ds-input`.
3. Keep existing data fetching, API calls, and state logic unchanged unless asked.

## Converting existing pages

1. Read target file + `COMPONENT-MAP.md`.
2. Replace visual markup/classes only; preserve handlers, routes, Prisma/API usage.
3. Sidebar/layout: align with preview `.nav` / card patterns.
4. Verify focus states use `--ds-focus-blue`.

## Stack

- Next.js Pages Router, React, Tailwind v4.
- Global styles: `src/styles/globals.css` imports `Design/vercel/tokens.css`.

When unsure, open `Design/vercel/preview.html` in browser and match it — do not guess colors.

---
> Source: [manzis/invobook](https://github.com/manzis/invobook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
