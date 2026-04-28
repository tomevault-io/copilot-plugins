## jazz

> A minimal CSS design system. One stylesheet, no build step required for users.

# Jazz — Agent Guide

A minimal CSS design system. One stylesheet, no build step required for users.

## Repo structure

```
packages/
  core/      — The CSS library (source + build)
  docs/      — Documentation site (Hono SSG + Vite)
main.css     — Built output, committed to repo
```

## Dev commands

```bash
pnpm dev        # Run core + docs in parallel (watch mode)
pnpm build      # Build core CSS + generate static docs
```

From `packages/docs`:

```bash
pnpm dev        # Vite dev server with HMR
pnpm build      # Build + generate all static HTML pages
```

From `packages/core`:

```bash
pnpm build      # Bundle src/main.css → ../../main.css (minified)
```

## Core CSS (`packages/core/src/`)

All styles live inside `@layer jazz { }`. This is important — it means any unlayered user styles always win without needing `!important`.

Key files:

- `src/main.css` — entry point, imports everything
- `src/colors.css` — color scales derived from seed variables
- `src/easings.css` — `--ease-glide`, `--ease-snap`, `--ease-heavy`
- `src/spacing.css` — `--jazz-spacing`, `--jazz-spacing-1` through `--jazz-spacing-8`
- `src/components/*.css` — one file per component

### Adding a component

1. Create `src/components/my-component.css` inside `@layer jazz { }`
2. Import it in `src/main.css`
3. Prefer styling native HTML elements or minimal class names (e.g. `.field`, `.switch`, `.badge`)
4. Avoid JavaScript — use CSS-only where possible (`:has()`, `@starting-style`, Popover API, anchor positioning)

### CSS conventions

- Colors: always use `var(--jazz-neutral-*)`, `var(--jazz-primary)`, etc. Never hardcode hex values
- Spacing: use `var(--jazz-spacing-*)` tokens
- Transitions: use `var(--ease-glide)` / `var(--ease-snap)` / `var(--ease-heavy)`
- Dark mode: handled automatically via `color-scheme` and the `.jazz-light` / `.jazz-dark` classes
- No em dashes in any text (copy, comments, docs)

## Docs site (`packages/docs/src/`)

Built with Hono (server-side rendering) + Vite. Pages are TypeScript files that return HTML strings using Hono's `html` tagged template literal.

Key files:

- `src/index.ts` — route definitions
- `src/layout.ts` — shared layout, sidebar nav, `components` array
- `src/pages/*.ts` — one file per page
- `src/docs.css` — docs-only styles (not part of the library)
- `scripts/generate.mjs` — generates static HTML for all routes

### Adding a docs page

1. Create `src/pages/my-page.ts` exporting an async function
2. Register the route in `src/index.ts`
3. Add the path to the routes array in `scripts/generate.mjs`
4. If it's a component page, add it to the `components` array in `src/layout.ts`
5. If it's a section page, add it to the sections nav in `src/layout.ts`

### Page structure

```ts
import { html, raw } from "hono/html";
import { Layout } from "../layout";
import { highlight } from "../highlight";

const toc = [{ id: "example", label: "Example" }];

export async function MyPage(path: string) {
  return Layout({
    title: "My Component",
    path,
    toc,
    content: html`
      <h1>My Component</h1>
      <p>Short description.</p>

      <h2 id="example">Example</h2>
      <div class="example">
        <div class="preview">
          <!-- live preview here -->
        </div>
        <div class="code-block">
          ${raw(await highlight(`<!-- code shown to user -->`))}
        </div>
      </div>
    `,
  });
}
```

### Example patterns

- `<div class="example">` — wraps a preview + code block pair
- `<div class="preview">` — renders the live component
- `<div class="preview preview-padded">` — left-aligned, column layout
- `<div class="code-block">` — syntax-highlighted code (use `highlight()`)
- `highlight(code, lang?)` — Shiki syntax highlighter, defaults to HTML

## Generated files

These files are regenerated automatically on every `pnpm build` — do not edit them by hand:

- **`SKILL.md`** — generated from `packages/docs/src/pages/skill.ts`. Injects the component list and CDN URL from source. Edit the template in `skill.ts` instead.
- **`dist/llms.txt`** — generated from `packages/docs/src/pages/llms.ts`.

## Design principles

- Jazz styles native elements directly. No class names needed for basic usage
- UI only, not layout. Jazz does not provide grid/flex utilities
- Semantic HTML first. Use the right element for the job
- One file. The entire library ships as a single `main.css`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erikthalen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
