## r00t-kim-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio site for a security researcher. Built with Next.js App Router, statically exported to GitHub Pages at `https://r00t-kim.github.io`.

## Commands

```bash
npm run dev      # Local dev server
npm run build    # Static export to ./out/
npm run lint     # ESLint
```

No test suite is configured.

## Architecture

### Static Export

`output: "export"` in `next.config.ts` — no server-side runtime, no API routes, images unoptimized. All pages pre-rendered at build time via `generateStaticParams`. The `./out/` directory is deployed to GitHub Pages via GitHub Actions.

### Routing

```
/ → redirect to /ko
/[locale]         → main portfolio page (all sections)
/[locale]/career  → career story detail page
```

Locales: `ko` (default), `en`. Configured in `src/i18n/routing.ts`.

### Bilingual Content Pattern

Two separate systems for translations:

- **Data content** (`src/data/*.ts`): Every object carries sibling fields like `title` / `titleEn`. Components check `locale === "ko"` and select the appropriate field directly.
- **UI labels** (`src/messages/{ko,en}.json`): Navigation, section headings, button text — accessed via next-intl's `useTranslations()`.

### Styling

Tailwind CSS v4 via PostCSS plugin — **no `tailwind.config.js`**. Theme tokens (colors, fonts, spacing) defined in `src/app/globals.css` using `@theme { }` block with CSS custom properties. Dark mode via `.dark` class (next-themes, default: dark). Accent color: `--color-accent` (emerald `#10b981`).

### Provider Tree

```
ThemeProvider (next-themes)
  → NextIntlClientProvider
    → ErrorBoundary
      → LazyMotionProvider (Framer Motion domAnimation)
```

Defined in `src/app/[locale]/layout.tsx`.

### Key Conventions

- **Path alias**: `@/*` → `src/*`
- **Class utility**: `cn()` from `src/lib/utils.ts` (clsx + tailwind-merge)
- **All section components** are `"use client"` with Framer Motion `whileInView` scroll animations
- **No state management library** — local `useState` only
- **Site config**: `src/config/site.ts` is the single source for URL, author info, social links, nav keys, SEO metadata

### Content Editing Map

| What | File |
|------|------|
| Personal info, education | `src/data/profile.ts` |
| Work/research experience | `src/data/experience.ts` |
| Career highlights, competencies | `src/data/career-summary.ts` |
| Career story narratives | `src/data/career-detail.ts` |
| Projects | `src/data/projects.ts` |
| Certifications, awards | `src/data/credentials.ts` |
| Talks, community | `src/data/speaking.ts` |
| Site URL, author, nav, SEO | `src/config/site.ts` |
| UI labels (Korean) | `src/messages/ko.json` |
| UI labels (English) | `src/messages/en.json` |
| Colors, fonts, theme tokens | `src/app/globals.css` |

---
> Source: [R00T-Kim/r00t-kim.github.io](https://github.com/R00T-Kim/r00t-kim.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
