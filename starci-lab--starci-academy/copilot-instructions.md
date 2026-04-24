## starci-academy

> Next.js App Router + React frontend conventions for starci-academy (components, pages, data, styling).


# Frontend Conventions (StarCI Academy)

Stack: **Next.js** (App Router), **React 19**, **Tailwind CSS v4**, **HeroUI** (`@heroui/react`), **next-themes**, **SWR**, **Phosphor Icons**, **Framer Motion** where needed.

---

## Folder roles

| Area | Path | Purpose |
|------|------|--------|
| Routes & layouts | `src/app/` | `page.tsx`, `layout.tsx`, route segments, route groups. Keep route files thin; move UI into `src/components/`. |
| UI | `src/components/` | Reusable components, layouts, providers. Feature folders with `index.tsx` or named entry. |
| Static course / marketing content | `src/data/` | Course modules, copy, structured content (often `.tsx` when content includes JSX). |
| View / page-level types | `src/types/` | Types shaped for the UI (e.g. course cards, pricing), not API entity mirrors. |
| Shared API / domain types | `src/modules/types/` | Schemas aligned with backend contracts; import from `@/modules/types` when typing API responses. |

Do **not** put large presentational trees only used in one route under `app/` long-term; prefer `components/` and import.

---

## App Router (`src/app/`)

- **`page.tsx`** — Server Component by default unless the file or a child needs `"use client"`.
- **`layout.tsx`** — Shell, fonts, global providers composition. Prefer wrapping providers from `src/components/providers/`.
- **Colocate** route-only helpers next to the route (e.g. small `loading.tsx`, `error.tsx`) when used only there.
- **Dynamic routes** — Use descriptive param names in folders (e.g. `[id]`); type `params` / `searchParams` from Next.js types where applicable.

---

## Components (`src/components/`)

- **One main concern per folder** — e.g. `layouts/Course/`, `layouts/Navbar/`, `providers/`.
- **Public API** — Re-export from folder `index.ts` when the folder is a package of related pieces (match existing `layouts`, `providers` pattern).
- **File naming** — `PascalCase` for component files is acceptable; `index.tsx` inside a feature folder is fine for the primary export.
- **Props** — Prefer a single `Props` type or `{ComponentName}Props` interface in the same file (or adjacent `types.ts` if large). **No nested object types inline** in props: extract sub-interfaces (same rule as backend Kani types).
- **Default export** — Use default export for the primary page-level or layout component in `app/` if that matches the file; in `components/`, default or named exports are both OK but **stay consistent within a folder**.

---

## `"use client"` and boundaries

- Add **`"use client"`** only at the boundary that needs browser APIs, state, effects, event handlers, or client-only libraries.
- Keep **Server Components** as the default for static content, SEO-friendly markup, and data that can be passed as props from the server.
- Do **not** mark the entire tree as client-only; push `"use client"` down to the smallest subtree.

---

## Styling (Tailwind + HeroUI)

- Use **Tailwind utility classes** for layout, spacing, typography, and responsive behavior.
- Use **HeroUI** primitives for interactive controls (buttons, inputs, modals, etc.) for consistency with the design system.
- Theme extension lives in **`src/app/hero.ts`** (HeroUI plugin). Prefer design tokens / component variants over hard-coded one-off colors when HeroUI provides them.
- Prefer **`cn()` / conditional classes** patterns only if the project already introduces a helper; otherwise use template literals or `clsx`-style utilities if added later—**do not add new deps without need**.

---

## Data & content (`src/data/`)

- Group by **course or product** (e.g. `fullstack/`, `system-design/`) with **`modules/`** for ordered lessons.
- **`index.ts`** files re-export the public surface for that course.
- Keep **IDs and slugs** stable when content is linked from routes or `src/types`.

---

## Types

- **`src/types/`** — UI-facing shapes (forms, cards, tables). Comment non-obvious fields with short `//` or JSDoc.
- **`Array<T>`** for array types in shared type files (align with backend Kani rule) unless the file already consistently uses `T[]`.
- **`import type`** for type-only imports in TS/TSX.

---

## Imports

- Use the **`@/`** path alias (`@/components/...`, `@/types/...`, `@/modules/...`).
- Order: external packages → `@/` → relative (group logically). Type-only imports grouped as in the backend rule.

---

## Icons & motion

- **Icons:** `@phosphor-icons/react`; import only the icons used (tree-shaking friendly).
- **Motion:** `framer-motion` only where animation is required; avoid wrapping large static regions.

---

## Data fetching (SWR)

- Configure SWR via **`src/components/providers/SwrProvider.tsx`** (or the project’s single provider). Do not duplicate global SWR config in every hook.
- Custom hooks that wrap `useSWR` live next to the feature or under a `hooks/` folder if the project introduces one—**one hook per file** for non-trivial logic.

---

## Comments (mirror backend spirit)

- **Components:** A short top-of-file or above-export comment if behavior is non-obvious (e.g. client boundary, accessibility notes).
- **Complex handlers:** Use `//` between major steps (validate → call API → update UI state).
- **Public hooks / utilities:** JSDoc with `@param` / `@returns` when the API is reused.

---

## Accessibility & semantics

- Prefer **semantic HTML** (`main`, `nav`, `article`, headings in order).
- Interactive elements must be **keyboard reachable**; use HeroUI/native controls instead of `div` with `onClick` unless there is a clear a11y pattern.

---

## ESLint

1. Read **`eslint.config.mjs`** (or project ESLint config) before large style changes.
2. Run **`npm run lint`** and fix issues so changes pass the project’s rules.

Code must pass ESLint in addition to the conventions above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/starci-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
