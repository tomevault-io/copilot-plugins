## sudops-pl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Keep this file up to date as the project evolves.** When you add features, change structure, or learn something new about the codebase, update CLAUDE.md in the same commit — don't wait to be asked.

## Project Overview

Personal blog and portfolio site (sudops.pl) built with **Astro 6** using the astro-erudite template. Uses MDX for content, Tailwind CSS 4 for styling, shadcn/ui for UI components, and React for interactive components.

## Deployment

The site is hosted on **Cloudflare Pages**, connected directly to this repo. Every commit pushed to `main` triggers an automatic build and deploy. Results are live at **https://sudops.pl**.

To build, commit, and deploy in one shot:
```
npm run build && git add . && git commit -m "your message" && git push origin main
```

You can verify the result at https://sudops.pl after the Cloudflare build completes.

### Cloudflare Environment Variables
- `BREVO_API_KEY` — Brevo API key for newsletter
- `BREVO_LIST_ID` — Brevo mailing list ID

## Commands

- `npm run dev` — Start dev server (port 1234)
- `npm run build` — Type-check (`astro check`) then build
- `npm run preview` — Preview production build
- `npm run prettier` — Format all source files

## Architecture

**Path alias:** `@/*` maps to `./src/*`

### Content System (`src/content/`)
- **Blog posts** (`blog/`): MDX files with frontmatter (title, description, date, tags, image, authors, draft, order). Subposts use directory nesting — a post ID containing "/" is a subpost (e.g., `parent-slug/child-slug`).
- **Authors** (`authors/`): Markdown profiles referenced by blog posts.
- **Projects** (`projects/`): Markdown with name, description, tags, image, link, dates.

Content schemas are defined in `src/content.config.ts`.

### Key Files
- `src/consts.ts` — Site metadata, nav links, social links, newsletter consent text
- `src/lib/data-utils.ts` — All content querying functions (posts, tags, subposts, TOC, reading time)
- `src/styles/global.css` — Theme color variables (OKLCH format), light/dark mode via `data-theme` attribute
- `src/styles/typography.css` — Prose/blog content typography (Geist font family)
- `astro.config.ts` — Astro config with MDX, React, sitemap, expressive-code, rehype/remark plugins

### Component Patterns
- Astro components in `src/components/` for static rendering (Header, Footer, BlogCard, ProjectCard, Callout, TOC, etc.)
- React components in `src/components/react/` for client-side interactivity (Newsletter, NewsletterPopup, ErrorBoundary)
- shadcn/ui primitives in `src/components/ui/` (avatar, badge, button, dialog, pagination, etc.)
- `src/components/Callout.astro` has 22 callout variants (note, tip, warning, danger, theorem, etc.)
- `src/components/BomPieChart.tsx` — interactive recharts doughnut chart (used in BOM post)

### Newsletter Integration
- **Popup form** (`src/components/react/NewsletterPopup.tsx`) — floating mail button + auto-popup after 40% scroll, dismissible per session
- **Form** (`src/components/react/Newsletter.tsx`) — email input with GDPR consent checkbox
- **API** (`functions/api/newsletter/subscribe.ts`) — Cloudflare Pages Function, adds contacts to Brevo via `/v3/contacts` API
- **Confirmation page** (`src/pages/newsletter/confirmed.astro`)
- Campaigns/newsletters sent manually via Brevo UI

### Styling
- Tailwind CSS 4 with `@tailwindcss/vite` plugin
- Theme colors defined as CSS custom properties in OKLCH color space
- Light/dark mode toggled via `data-theme` attribute on `<html>`

### Blog Structure

The blog is a homelab series. Posts use a parent/subpost pattern where topics with multiple parts get a parent `index.mdx` (overview + callout listing subposts) and individual subpost `.mdx` files with `order` frontmatter:

- `homelab-why/` — Standalone post (why this project)
- `homelab-design/` — Parent + subposts: `compute.mdx`, `network.mdx`, `storage.mdx`
- `homelab-bom/` — Parent + subposts: `pre-validation.mdx`, `post-sno.mdx`
- `homelab-network-impl/` — Parent + subposts: `phase0.mdx`, post-SNO *(future)*
- `homelab-validation/` — Parent + subposts: `hardware.mdx`, `sno.mdx`
- `homelab-day1/` — Standalone post (OKD 3-node cluster installation)
- `homelab-day2/` — Parent + subposts: `bootstrap.mdx`, `cert-manager.mdx`, `storage-network.mdx`, `rook-ceph.mdx` *(future)*

When converting raw drafts to MDX:
- Strip social media drafts (LinkedIn/Slack/Reddit) from the end
- Convert blockquotes/notes to `<Callout>` components with appropriate variants
- Use ` ```routeros ` for MikroTik/RouterOS config blocks (custom TextMate grammar in `src/grammars/`), `bash` for pure shell commands, plain ` ``` ` for terminal output with prompts
- Images go in the same directory as the post, referenced with `./filename.png`
- Use first person ("I"), never "we" — this is a personal blog
- Keep prose tight and conversational. Avoid textbook-style explanations — assume the reader has context. If something can be said in one sentence, don't use three.
- Prefer simple words over fancy ones (e.g. "leftovers" not "remnants", "locked down" not "walled garden", "can't be changed" not "immutable", "removes" not "eliminates")
- **MDX gotcha:** `<` followed by a digit (e.g. `<100 ms`, `<1 ms`) gets parsed as a JSX tag and breaks the build. Use "under 100 ms" or "less than 1 ms" instead.
- Summary diagrams (e.g. `hwvalidation.png`, `snovalidation.png`) go after the go/no-go table at the end of validation posts, before "What's next"

### Pages
- `/blog` — Blog listing (10 posts per page, grouped by year, minimal card style)
- `/privacy` — Privacy policy (GDPR, Brevo, Cloudflare)
- `/terms` — Terms & conditions
- `/newsletter/confirmed` — Post-subscription confirmation
- `/tags` — Tag index
- `/projects` — Project portfolio

### Raw Blog Post Drafts

Raw/unformatted post drafts are stored in `posts/*.md` (gitignored). These need to be formatted (proper frontmatter, MDX syntax, callouts, etc.) and moved to `src/content/blog/` before they go live.

### Reference Content (`test-content/`)

The `test-content/` directory contains the original astro-erudite template content. **Always use it as a reference** when creating new content to ensure correct structure:
- `test-content/authors/` — Author profile format (frontmatter fields: name, avatar, bio, socials)
- `test-content/blog/` — Blog post structure (directory per post, `index.mdx` + optional `banner.png`, subpost nesting)
- `test-content/projects/` — Project entry format (frontmatter fields: name, description, tags, image, link)

When formatting raw drafts from `posts/`, consult these examples for proper frontmatter schema, MDX conventions, image handling, and subpost directory layout.

### Prettier Config
Semi-colon free, single quotes. Plugins: astro, tailwindcss, astro-organize-imports.

---
> Source: [sudoom/sudops.pl](https://github.com/sudoom/sudops.pl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
