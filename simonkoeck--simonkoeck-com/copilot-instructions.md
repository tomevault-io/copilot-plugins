## simonkoeck-com

> Personal portfolio and security vulnerability writeup website for Simon Koeck (simonkoeck.com).

# CLAUDE.md

## Project Overview

Personal portfolio and security vulnerability writeup website for Simon Koeck (simonkoeck.com).

## Architecture

Turborepo monorepo:

- **apps/web** — Astro site (portfolio + writeups)
- **packages/typescript-config** — Shared tsconfig presets

## Tech Stack

- **Framework:** Astro 6 with experimental Rust compiler
- **Hosting:** Cloudflare Workers (`@astrojs/cloudflare` adapter)
- **Content:** MDX via `@astrojs/mdx`, Astro Content Collections (glob loader)
- **Styling:** Tailwind CSS 4 (via Vite plugin) + `@tailwindcss/typography`
- **Syntax Highlighting:** Shiki (built into Astro, github-dark theme)
- **Search:** Pagefind (client-side full-text search, indexed at build time)
- **Language:** TypeScript 5.9 (strict)
- **Package Manager:** pnpm 9
- **Build:** Turborepo 2.9
- **CI/CD:** GitHub Actions -> Cloudflare Workers

## Commands

```sh
pnpm dev          # Start dev server
pnpm build        # Build all workspaces
pnpm lint         # Lint all workspaces
pnpm format       # Format with Prettier (ts, tsx, md)
pnpm check-types  # TypeScript / astro check
```

In `apps/web`:

```sh
pnpm deploy       # Build + deploy to Cloudflare Workers
pnpm preview      # Local preview via wrangler dev
```

## Content

Writeups live in `apps/web/src/content/writeups/` as `.mdx` files with typed frontmatter:

```yaml
title: string
date: date
severity: info | low | medium | high | critical
cvss: number (0-10, optional)
platform: string
tags: string[]
summary: string
cve: string (optional, e.g. CVE-2026-1234)
ghsa: string (optional, e.g. GHSA-abcd-efgh-ijkl)
published: boolean (default: true)
```

Schema defined in `apps/web/src/content.config.ts` using Astro 6 glob loader.

Set `published: false` to hide a writeup from the site (draft mode).

## MDX Components

Available for use in writeups:

- `<Callout type="info|warning|danger|tip">` — colored aside with left border accent
- `<Timeline events={[{date, title}]}/>` — vertical disclosure timeline
- `<HttpRequest method="GET" url="..." headers={{}} body=""/>` — HTTP request display

## Writing Style

Writeups should be written in natural language, not dry vulnerability reports. Show technical depth (PoCs, code snippets, request/response details) but also tell the story of how the vulnerability was actually found. Walk through the thought process, what caught your attention, what you tried, and how you got from recon to confirmed impact. The reader should feel like they're following along with the discovery, not reading a template.

Never use em dashes (—). Use commas, periods, or parentheses instead.

When creating a new writeup, always set the `date` field to today's date.

## Conventions

- Pages in `apps/web/src/pages/` (file-based routing)
- Layouts in `apps/web/src/layouts/` (Base.astro, Writeup.astro, Page.astro)
- Markdown pages (imprint, privacy) use `Page.astro` layout
- Tailwind configured via CSS in `apps/web/src/styles/global.css` (Tailwind v4 CSS-first config)
- Severity colors defined as `--color-severity-*` theme tokens
- Dark-only design (zinc-950 background)
- Vim keybindings enabled globally (j/k navigation, / for search, gg/G)
- Command palette via Cmd+K with Pagefind full-text search
- All external links get an external icon via CSS `::after`
- OG images auto-generated via satori/resvg (prerendered in Node.js)

---
> Source: [simonkoeck/simonkoeck.com](https://github.com/simonkoeck/simonkoeck.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
