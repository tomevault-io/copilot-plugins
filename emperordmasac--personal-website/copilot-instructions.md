## personal-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start the Next.js dev server at http://localhost:3000
- `npm run build` — production build
- `npm run start` — serve the production build
- `npm run lint` — run `next lint`

There is no test suite configured.

## Architecture

Single-page personal resume/portfolio site built with the Next.js 14 App Router. The entire page is rendered from one route (`app/page.tsx`) driven by a single source of truth.

### Data flow

All resume content (name, about, skills, projects, work, education, social links) lives in `data/resume-data.tsx` as a `const`-asserted `RESUME_DATA` object. `app/page.tsx` imports it and maps over its arrays to render sections; `app/layout.tsx` also reads it to build the `<title>` metadata. To change anything visible on the site — copy, ordering, projects, jobs, skills — edit `resume-data.tsx`, not the page component.

Images referenced from `RESUME_DATA` (project logos, screenshots, school logos) are imported as static assets through `public/index.ts`, which re-exports every image in `public/` under a named binding. New images must be added to that barrel file before they can be referenced from the data file via `@/public`.

### UI layer

- shadcn/ui (style: "new-york", base color: zinc) configured in `components.json`. Primitives live in `components/ui/` and are imported via the `@/components/ui/*` alias.
- Tailwind is theme-driven via CSS variables defined in `app/globals.css`; colors should be referenced through the semantic tokens declared in `tailwind.config.ts` (`background`, `foreground`, `primary`, `muted`, etc.) rather than raw values.
- `cn()` from `lib/utils.ts` (clsx + tailwind-merge) is the standard class-merging helper.
- Theming uses `next-themes` via `components/theme-provider.tsx`, wired in `app/layout.tsx`. `defaultTheme` is `"light"`.
- The page is print-styled: classes like `print:hidden`, `print:flex`, and `print:p-12` exist throughout `app/page.tsx` so the site doubles as a printable resume. Preserve these when editing layout.

### Interactive pieces

- `components/command-menu.tsx` is a client component (`cmdk`-based) that opens with ⌘K / Ctrl+K and lists social links. It is mounted at the bottom of `app/page.tsx` and fed `RESUME_DATA.contact.social`.
- Social icons live in `components/icons/` and are referenced from `resume-data.tsx`. Add new ones there and re-export through `components/icons/index.ts`.

### Path aliases

`@/*` maps to the project root (see `tsconfig.json`). Use `@/components`, `@/lib/utils`, `@/data/resume-data`, and `@/public` for imports rather than relative paths.

### Analytics

`@vercel/analytics/react`'s `<Analytics />` is mounted in `app/layout.tsx`. The site is intended to deploy on Vercel.

---
> Source: [Emperordmasac/personal-website](https://github.com/Emperordmasac/personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
