## nsc-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # start dev server at localhost:4321
npm run build    # build to dist/
npm run preview  # preview the production build locally
```

There are no tests or linters configured.

## Architecture

This is an **Astro 6 static site** with **Tailwind CSS 4** (via `@tailwindcss/vite`). Content is managed as Markdown files using Astro's content collections with the glob loader (defined in `src/content.config.ts`, not `src/content/config.ts`).

### Content collections

Three collections, each a folder of `.md` files with YAML frontmatter:

| Collection | Folder | Key fields |
|---|---|---|
| `students` | `src/content/students/` | `name`, `batch_start`, `batch_end?` |
| `projects` | `src/content/projects/` | `type` (thesis/short/other), `student` (slug ref), `supervisors[]` |
| `posts` | `src/content/posts/` | `author` (student slug ref), `date` |

`student` and `author` fields are plain strings that reference a student's slug (filename without `.md`) — they are **not** Astro `reference()` relations, just manual joins done with `getCollection` + `find`/`filter` in each page.

### Pages and routing

- `/` — student grid with client-side batch year filter
- `/students/[slug]` — profile + that student's projects + posts
- `/projects` — all projects with client-side type/year filter + text search
- `/blog` — aggregated post feed
- `/blog/[slug]` — full post with rendered markdown
- `/admin/` — Decap CMS UI (static HTML in `public/admin/`)

### Styling

Tailwind 4 with custom theme tokens defined in `src/styles/global.css` using `@theme {}`. Key tokens: `--color-eth-red: #E3000B`, `--color-ink`, `--color-surface-alt`, `--color-border`, `--color-border-bold`. Use these as `text-eth-red`, `border-border-bold`, etc. Prose styling for blog post bodies uses the `.prose` class defined in `global.css` (no Tailwind Typography plugin).

Design is **Swiss/Bauhaus**: sharp corners (no rounded borders on cards), bold black/red typography, strong grid layout.

### Decap CMS

`public/admin/config.yml` is configured but not yet active. To activate: enable Netlify Identity + Git Gateway on the Netlify dashboard, then replace `OWNER/REPO` in `config.yml` with `jjjoelle/nsc_website`. Editorial workflow is on — new content creates PRs rather than publishing directly.

### Deployment

Hosted on Netlify at `https://snazzy-sprinkles-b80bb1.netlify.app`. GitHub repo: `jjjoelle/nsc_website`. Netlify auto-deploys on push to `main`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jjjoelle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
