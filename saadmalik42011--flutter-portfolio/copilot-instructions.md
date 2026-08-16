## flutter-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Premium, dark-first portfolio site for a Lead Mobile Developer. Next.js 16 (App Router), TypeScript, Tailwind CSS v4, Framer Motion, GSAP + Lenis. Not currently a git repository.

## Commands

```bash
npm install
npm run dev      # start dev server at http://localhost:3000
npm run build    # production build
npm run start    # serve production build
npm run lint     # eslint (flat config via eslint.config.mjs)
```

There is no test suite/runner configured in this repo.

## Content model — read this before touching sections/components

**Every word and number on the site lives in `data/*.json`.** Components never hardcode content — they import typed accessors from `src/lib/data.ts` and render whatever the JSON provides. When asked to change site copy, stats, projects, skills, etc., edit the JSON file, not the component.

- `src/types/content.ts` — the typed schema for every JSON file (one interface per data file/shape).
- `src/lib/data.ts` — imports each `data/*.json`, casts it to its type, and re-exports it (`personal`, `hero`, `about`, `achievements`, `technologies`, `skills`, `experience`, `projects`, `services`, `testimonials`, `education`, `certifications`, `blogPosts`, `footer`, `navigation`, `social`) plus a few derived helpers (`featuredProjects`, `getProject(slug)`, `getBlogPost(slug)`, `techCategories`).
- Adding a field to a data file requires updating the matching interface in `content.ts` too — components consume the typed shape, not the raw JSON.
- Any `"icon"` string in JSON refers to a lucide-react icon name and must be registered once in `src/lib/icons.tsx`'s `registry` (plus `Github`/`Linkedin` are hand-drawn inline SVGs since lucide dropped brand glyphs). Use `<Icon name="..." />` or `getIcon(name)` to resolve — unregistered names silently fall back to `Sparkles`.
- Adding a project: append to `data/projects.json` (every field documented by example in existing entries); optionally add `public/projects/<slug>/` images referenced by `cover`/`screenshots`. If `cover` is unset, `src/components/projects/project-visual.tsx` auto-generates a stylised phone mock from the project's `accent` color, `icon`, and `stats` — no screenshots needed for a new project to look finished.
- Rebuilding after a JSON edit regenerates cards, showcase rows, detail pages, case studies, sitemap, and OG tags automatically — there's no separate registration step.

## Architecture

```
data/                 ← ALL content (JSON) — see Content model above
public/resume/        ← resume PDF (filename referenced by data/personal.json → resumeUrl)
public/projects/      ← project images (referenced from JSON)
src/
  types/content.ts    ← typed schema for every JSON file
  lib/data.ts         ← typed data access (see above)
  lib/icons.tsx       ← lucide icon registry
  lib/utils.ts        ← cn() = clsx + tailwind-merge
  components/
    ui/               ← button, badge, glow-card, inputs, section heading
    effects/          ← reveal, aurora, particles, spotlight, magnetic, tilt, counter
    layout/           ← navbar, footer, section, theme toggle, back-to-top
    hero/             ← hero + signature phone showcase
    sections/         ← about, tech-stack, experience, projects, skills, services, testimonials, blog, contact, achievements
  app/                ← routes + SEO (sitemap.ts, robots.ts, manifest.ts, opengraph-image.tsx)
```

- `src/app/page.tsx` composes the whole homepage as a flat sequence of section components (`Hero`, `About`, `Achievements`, `TechStack`, `Experience`, `Projects`, `Skills`, `Services`, `Testimonials`, `Blog`, `Contact`) — reordering the page means reordering that list.
- `src/app/layout.tsx` wires up fonts (Inter/Manrope/JetBrains Mono via CSS variables), global `Metadata`/JSON-LD (derived from `data/personal.json` + `data/social.json`), and wraps the tree in `Providers` (theme + smooth scroll) with global `Navbar`/`Footer`/`Spotlight`.
- `src/components/providers/providers.tsx` sets up `next-themes` (`data-theme` attribute, dark default, no system detection) and a Lenis smooth-scroll instance synced to GSAP's `ScrollTrigger`/ticker; smooth scroll is skipped when `prefers-reduced-motion` is set. Use `useLenis()` to access the instance elsewhere.
- Theming: CSS custom properties defined in `:root` and `[data-theme="dark"]` in `src/app/globals.css`, exposed to Tailwind v4 via `@theme inline`. Dark is the default theme; toggled by `next-themes` (see `layout/theme-toggle.tsx`). Custom keyframe-based animation utilities (`animate-float`, `animate-aurora`, `animate-shimmer`, etc.) are also declared here.
- Animation conventions: `components/effects/reveal.tsx` provides `Reveal` (single scroll-triggered fade/rise/blur-in, respects `useReducedMotion`) and `Stagger`/`StaggerItem` (staggered group entrance). Most section components wrap content in these rather than writing raw Framer Motion variants.
- Dynamic routes `app/projects/[slug]/page.tsx` and `app/blog/[slug]/page.tsx` look up content via `getProject`/`getBlogPost` and should call `notFound()` for unknown slugs; sitemap/OG generation in `app/` iterates the same `projects`/`blogPosts` arrays.
- Path alias `@/*` maps to `src/*` (see `tsconfig.json`).

---
> Source: [saadmalik42011/Flutter_Portfolio](https://github.com/saadmalik42011/Flutter_Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
