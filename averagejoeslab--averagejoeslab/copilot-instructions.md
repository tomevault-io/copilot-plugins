## averagejoeslab

> Average Joes Lab is an open research platform for citizen scientists. This website is built with Next.js 16, React 19, Tailwind CSS 4, and TypeScript. It deploys as a static export to GitHub Pages at `averagejoeslab.com`.

# AGENTS.md - Average Joes Lab Repository Guide

## Overview

Average Joes Lab is an open research platform for citizen scientists. This website is built with Next.js 16, React 19, Tailwind CSS 4, and TypeScript. It deploys as a static export to GitHub Pages at `averagejoeslab.com`.

## Tech Stack

- **Framework**: Next.js 16 with static export (`output: 'export'`)
- **Styling**: Tailwind CSS 4 with `@theme` custom properties
- **Language**: TypeScript (strict mode)
- **Markdown**: unified pipeline (remark-parse → remark-gfm → remark-rehype → rehype-raw → rehype-mermaid → rehype-stringify)
- **Icons**: lucide-react
- **Hosting**: GitHub Pages with custom domain (averagejoeslab.com)

## Commands

```bash
npm run dev          # Start dev server (fetches blog content first)
npm run build        # Build static export + generate sitemap
npm run lint         # ESLint check
npm run type-check   # TypeScript check (tsc --noEmit)
npm run fetch-content # Clone blog repo into content/blog/
```

## Project Structure

```
app/                    # Next.js App Router pages
  layout.tsx            # Root layout (Header + Footer)
  page.tsx              # Homepage
  not-found.tsx         # 404 page
  blog/                 # Blog listing and [slug] pages
  internal-papers/      # Internal research papers
  external-papers/      # External papers & reproductions
  learn/                # Learning paths index and [...slug] pages
components/             # React components
  Header.tsx            # Sticky nav with mobile menu ('use client')
  Footer.tsx            # Site footer (server)
  JsonLd.tsx            # JSON-LD schema helper (server)
  BlogPostGrid.tsx      # Blog tag filtering ('use client')
  LearnSidebar.tsx      # Docs sidebar nav ('use client')
  InternalPapersContent.tsx  # Papers filtering/sorting ('use client')
  ExternalPapersContent.tsx  # Papers filtering/sorting ('use client')
  TypeformEmbed.tsx     # Typeform loader ('use client')
lib/                    # Server-side utilities
  blog.ts               # Blog markdown processing
  learn.ts              # Docs markdown processing + sidebar data
  papers.ts             # Paper types and color maps
data/                   # Notion-synced JSON (committed)
  internal-papers.json
  external-papers.json
content/
  learn/                # Docs markdown (committed)
    research-engineering/
    foundations/
  blog/                 # Blog content (gitignored, fetched at build)
scripts/
  fetch-content.sh      # Clone averagejoeslab-blog
  generate-sitemap.mjs  # Post-build sitemap generator
public/                 # Static assets
  CNAME, .nojekyll, robots.txt, llms.txt
  images/               # Logos and favicon
```

## Content Architecture

- **Blog posts**: Stored in separate `averagejoeslab/averagejoeslab-blog` repo, cloned at build time into `content/blog/`
- **Learning docs**: Committed in `content/learn/` with frontmatter (`title`, `description`, `sidebar_position`)
- **Papers data**: JSON files in `data/`, auto-synced from Notion databases daily via GitHub Action

## Color Palette

- **Slate** (primary neutral): 50 `#f7fafc` → 900 `#171923`, base 700 `#2d3748`
- **Accent** (blue): 50 `#ebf8ff` → 900 `#1a365d`, base 400 `#4299e1`

## Key Patterns

- Server components by default; `'use client'` only for interactive components
- All pages pre-rendered at build time (static export)
- Blog slugs derived from filenames: `2025-01-15-slug-title.md` → `/blog/slug-title/`
- Learn slugs from directory/filename: `content/learn/foundations/getting-started.md` → `/learn/foundations/getting-started/`
- Papers pages: server wrapper imports JSON, passes to client content component

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/averagejoeslab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
