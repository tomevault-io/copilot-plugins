## nostem-github-io

> This file gives runtime agents enough context to manage the site safely and consistently.

# Website Agent Context (nostem.github.io)

This file gives runtime agents enough context to manage the site safely and consistently.

## Mission

Maintain and evolve the website at `Nostem/nostem.github.io` with a PR-first workflow.

Common tasks:

- Add/edit books in bookshelf
- Add/edit writing content
- Create and update pages
- Keep navigation and UX consistent
- Validate changes before publishing

## Stack

- Next.js 14 (App Router)
- React 18 + TypeScript
- TailwindCSS + custom CSS variables
- Framer Motion for transitions
- Deployed to GitHub Pages via static export

## Build and deployment

- Static export is enabled in `next.config.mjs` (`output: 'export'`)
- GitHub Pages deploy action runs on `main` push
- Build artifact path is `./out`

## Repository map

- `src/app/page.tsx` - home page
- `src/app/books/page.tsx` - bookshelf page
- `src/app/books/[slug]/page.tsx` - book detail route
- `src/app/writing/page.tsx` - writing page
- `src/app/about/page.tsx` - about page
- `src/components/layout/Navigation.tsx` - global navigation links
- `src/components/layout/Footer.tsx` - footer links
- `src/data/books.ts` - bookshelf content data
- `src/lib/books.ts` - book utilities (grouping/filtering/slugs)
- `src/types/index.ts` - shared types

## Commands

```bash
npm ci
npm run lint
npm run build
npm run dev
```

## Required workflow (PR-first)

1. Create feature branch (`site/<task>-<date>`)
2. Implement requested change with minimal scope
3. Run `npm run lint` and `npm run build`
4. For route/UI changes, smoke-check key routes
5. Open PR with validation evidence
6. Never push directly to `main`

## Edit conventions

- Keep visual language consistent with existing typography and layout
- Preserve `PageTransition` usage on top-level pages where already used
- Avoid broad refactors when request is content-only
- Keep mobile nav/menu behavior working after nav updates

## Content model references

- `docs/content-model.md`
- `docs/publish-checklist.md`
- `docs/ops-commands.md`

## Safety constraints

- Do not change `.github/workflows/deploy.yml` unless explicitly requested
- Do not remove existing public routes without explicit approval
- Do not add heavy dependencies unless required by the request

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nostem)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Nostem)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
