## vibe-design-system

> Astro 6, Tailwind 4, daisyUI 5. Pre-scaffolded for building a design system and landing pages.

# Summer Vibes: Create Your Design System Using Cursor

Astro 6, Tailwind 4, daisyUI 5. Pre-scaffolded for building a design system and landing pages.

## Stack

| Piece | Location |
| --- | --- |
| Pages | `src/pages/` |
| Styles | `src/styles/global.css` (Tailwind + daisyUI plugin) |
| Dev server | `npm run dev` → http://localhost:4321 |

## Impeccable

```sh
npx impeccable skills install
```

Your design context lives in **`PRODUCT.md`** and **`DESIGN.md`** at the project root. Both are gitignored — local to this project, not shared via git.

## Design Context

**Do not hardcode design context in this file.** Read the local files below.

Before UI or design work:

1. Read `PRODUCT.md` if it exists (register, users, personality, anti-references, principles).
2. Read `DESIGN.md` if it exists (colors, typography, components).
3. If `PRODUCT.md` is missing, run `/impeccable init`. Do not infer brand from `README.md`.
4. Load context: `node .claude/skills/impeccable/scripts/context.mjs`

| File | Created by | Purpose |
| --- | --- | --- |
| `PRODUCT.md` | `/impeccable init` | Strategic design context |
| `DESIGN.md` | `/impeccable document` | Visual system from your code |

Impeccable commands (`/impeccable craft`, `/impeccable live`, `/impeccable critique`, etc.) read those files before doing work.

---
> Source: [lucas-datacamp/vibe-design-system](https://github.com/lucas-datacamp/vibe-design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
