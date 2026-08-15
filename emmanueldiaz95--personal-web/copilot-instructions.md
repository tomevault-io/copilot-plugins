## personal-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for Emmanuel Diaz — Finance Operations leader. Built with Next.js 16 (App Router) + TypeScript + Tailwind CSS. Monochromatic design system (black/grey/white) with dark/light mode toggle. Three pages: Experience (landing), Projects, Blog.

## Development Commands

- `npm run dev` — Dev server on localhost:3000
- `npm run build` — Production build (`.next/` server output)
- `npm run start` — Serve production build
- `npm run lint` — ESLint check

## Architecture

### Framework & Routing

Next.js 16 with App Router. File-based routing under `src/app/`:

- `/` — Experience page (landing) — stats, company cards
- `/projects` — Projects grid with search and CTA
- `/projects/[slug]` — Project detail page
- `/blog` — Blog with sidebar filters, categories, tags, newsletter
- `/blog/[slug]` — Blog post detail page

Built as a Next.js server app (`next build` → `.next/`, served with `next start`) and deployed on Railway. No static export.

### Component Organization

- `src/components/layout/` — Header (name masthead + social icons + theme toggle), PageTabs (sticky top navigation)
- `src/components/ui/` — Reusable: CompanyCard, ProjectCard, BlogPostCard, StatusBadge, TagPill, StatBlock, SearchBar, CTACard, NewsletterForm
- `src/components/ThemeToggle.tsx` — Dark/light mode toggle using next-themes

### Data Layer

No backend. Static TypeScript files under `src/data/`:
- `projects.ts` — Project array with typed `Project` interface, `getProjectBySlug()` helper
- `blogPosts.ts` — Blog post array with `getBlogPostBySlug()`, `getCategories()`, `getAllTags()` helpers
- `experience.ts` — Companies, stats, profile summary, earlier roles
- `constants.ts` — Contact info, social links, site metadata

### Styling & Theming

- Tailwind CSS with CSS custom properties for theming
- Color tokens defined in `src/app/globals.css` (`:root` for light, `.dark` for dark)
- `tailwind.config.ts` maps CSS variables to Tailwind color names
- `next-themes` with `class` strategy, default dark theme
- IBM Plex Mono (Google Fonts) for headings/labels/mono, system sans-serif for body

### Design System

- Monochromatic palette: no accent color, hierarchy from value contrast only
- Card system: `--surface` background, `--border` border, hover transitions, 10px radius
- Max-width: 1440px, responsive padding (64px → 40px → 20px)
- PageTabs: sticky segmented control (Experience/Projects/Blog) below header
- Social icons in header: LinkedIn, GitHub, Strava (lucide-react + custom SVG)

### Utilities

- `src/lib/utils.ts` — `cn()` classname helper for conditional class composition

---
> Source: [EmmanuelDiaz95/personal-web](https://github.com/EmmanuelDiaz95/personal-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
