## cameraman-landing

> This file provides guidance to AI assistants working on the Cameraman landing page.

# CLAUDE.md

This file provides guidance to AI assistants working on the Cameraman landing page.

## Project Overview

**Cameraman Landing** is the marketing site for [Cameraman](https://github.com/dPeluChe/cameraman), an open-source macOS screen recorder & editor. Deployed to [cameraman.dev](https://cameraman.dev) via Vercel.

- **Stack:** React 19 + TypeScript + Tailwind CSS 4 + framer-motion + Vite
- **Deploy:** Vercel (vercel.json configured)

## Build & Dev Commands

```bash
npm install
npm run dev      # http://localhost:3914
npm run build    # tsc -b && vite build → dist/
npm run lint     # eslint
npm run preview  # preview production build
```

## Architecture

```
src/
├── components/   Reusable UI: Nav, Footer, Button, Badge, SectionBackground,
│                 SectionHeading, Reveal (scroll animations), FeatureImage,
│                 EditorMockup (animated CSS editor preview)
├── sections/     Page sections: Hero, Why, Features, Agents, Comparison,
│                 Tech, Status, Install
├── lib/          cn.ts (class merge helper)
├── App.tsx       Assembles all sections
├── main.tsx      Entry point
└── index.css     Tailwind 4 @theme tokens + base styles
```

## Key Patterns

1. **Scroll animations:** Use `Reveal` for single elements, `Stagger` + `StaggerItem` for grids. Don't reinvent per-section.
2. **Section backgrounds:** Use `SectionBackground` with variants (`grid`, `mesh`, `orbs`, `dots`, `gradient`). Combinable.
3. **Feature status badges:** Must reflect the real state of the Cameraman app. Cross-reference `docs/TASK_TODO.md` and `docs/CHANGELOG.md` in the main repo.
4. **Screenshots:** `FeatureImage` accepts a `src` prop — when set, the gradient placeholder is replaced with the real image.
5. **Tailwind 4:** Custom theme tokens in `src/index.css` under `@theme`. Use `text-accent`, `bg-bg-card`, `border-border`, etc.
6. **TypeScript strict:** No `any`, no `@ts-ignore`.

## Content Sync

Stats in `sections/Tech.tsx` (LOC, test lines, MCP tool count, version) and feature status badges in `sections/Features.tsx` should be updated when the Cameraman app releases a new version. Source of truth:
- [Cameraman CHANGELOG](https://github.com/dPeluChe/cameraman/blob/main/docs/CHANGELOG.md)
- [Cameraman TASK_TODO](https://github.com/dPeluChe/cameraman/blob/main/docs/TASK_TODO.md)

## Related Repos

- **App repo:** [dPeluChe/cameraman](https://github.com/dPeluChe/cameraman) — the macOS app + EngineKit + MCP server
- **This repo:** [dPeluChe/cameraman-landing](https://github.com/dPeluChe/cameraman-landing) — this landing page

@~/.agents/skills/FLOW_CLAUDE.md

## ship config

```yaml
lint: npm run lint
typecheck: npx tsc -b --noEmit
build: npm run build
test: echo "no test suite defined"
merge_policy: ask
loc_limit: 500
simplify: 500
```

---
> Source: [dPeluChe/cameraman-landing](https://github.com/dPeluChe/cameraman-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
