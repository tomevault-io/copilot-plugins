## cpx-labs

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Node version

Use Node 22 (see `.nvmrc`). Native modules (`@next/swc`, `lightningcss`, `sharp`) crash on Node 24 in this WSL2 environment.

# Project structure

```
src/app/              # App Router pages
src/app/portfolio/    # Portfolio page — static project data array
src/components/       # Shared components (Header, Footer, etc.)
src/lib/              # Data modules (tools.ts) and utilities
public/portfolio/     # Puppeteer screenshots of project demos
openspec/             # Formal project specifications
scripts/              # Utility scripts (screenshots.mjs)
```

# Portfolio page data model

Projects are a static array in `src/app/portfolio/page.tsx`:

```typescript
interface Project {
  name: string;        // Display name
  slug: string;        // Screenshot filename (without extension)
  description: string; // PT-BR description
  repo: string;        // GitHub URL
  demo: string;        // Live demo URL
  tags: string[];      // Tech stack labels
}
```

Adding a new project: add entry to array + screenshot to `public/portfolio/{slug}.png`.

# Screenshot regeneration

```bash
node scripts/screenshots.mjs
```

Uses Puppeteer to capture 1280×800 viewport screenshots of each project's live demo. Output goes to `public/portfolio/`.

# Ferramentas (Tools) page

Tools live in `src/lib/tools.ts` (typed `Tool` array). The `/ferramentas` page renders them automatically — adding a tool entry is enough, no UI changes needed.

# Design tokens

Brand colors: `brand-50` through `brand-950` (purple theme). See `src/app/globals.css` for the full palette.

Tailwind CSS 4 with custom easing curves:
- `ease-spring` — bouncy spring animation
- `ease-out-expo` — exponential out for smooth transitions

---
> Source: [cpxlabs/cpx-labs](https://github.com/cpxlabs/cpx-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
