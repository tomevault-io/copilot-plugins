## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev        # Start dev server at localhost:4321
pnpm build      # Build production site to ./dist/
pnpm preview    # Preview the built site
pnpm resume     # Compile resume PDF using Typst
```

## Architecture

This is an Astro static site (TypeScript) for a personal portfolio with a Typst-based resume PDF.

### Data Flow

All portfolio content lives in a single `portfolio.toml` at the root. It is the source of truth for both the website and the PDF resume:

```
portfolio.toml
  → src/lib/config.ts (parsed at build time via smol-toml, exports typed config object)
  → src/components/*.astro (each section reads from config)
  → src/pages/index.astro (assembles components into the page)
  → resume/resume.typ (separate Typst file reads portfolio.toml for PDF output)
```

Individual items in `portfolio.toml` can be hidden without deletion by adding `enabled = false`.

### Toggling individual bullet points

Keep a large library of bullets per job and comment out the ones you don't want
for a given job description by prefixing the line with `#` inside the
`achievements` array:

```toml
achievements = [
  "Shown bullet.",
#  "Hidden bullet — commented out with a leading #.",
]
```

Nothing is deleted, so bullets can be switched on/off per job description. This
works for both the website and the resume since both read the same array.

### Keeping the resume to one page

The PDF resume (`pnpm resume`) must fit on a **single page**. When editing
`portfolio.toml`:

- Keep each bullet to **roughly one full line** — long enough to be substantive,
  short enough not to wrap to a second line at the resume's width.
- Enable only as many bullets as fit. Prefer commenting bullets **out** (leading
  `#`) over shortening strong ones. A good default is ~3 bullets for
  recent/major roles and 1–2 for older ones.
- After any content change, run `pnpm resume` and confirm the output is still one
  page. The PDF overflowing to a second page means too many bullets are active —
  comment some out rather than cramming.
- Bullets support `*word*` markup for **bold** (tech, metrics, product names);
  use it to emphasize impact, not to add length.


### Key Files

- `portfolio.toml` — all portfolio data (name, contact, jobs, projects, education, skills)
- `src/lib/config.ts` — TOML parser + config exporter; filters out disabled items
- `src/types/config.ts` — TypeScript types for the config shape
- `src/lib/bold.ts` — parses `*word*` syntax into `<strong>` tags for rich text fields

### Blog

Markdown posts go in `src/content/blog/`. Frontmatter requires `title` and `date` (YYYY-MM-DD); `description` is optional. The filename becomes the slug. Posts are sorted by date descending.

### Styling

Tailwind CSS v4 + DaisyUI v5, configured via `@tailwindcss/vite` plugin in `astro.config.ts`. No separate `tailwind.config.*` file — configuration is inline.

## Docs

The `docs/` folder contains developer documentation. Keep `docs/README.md` updated as a table of contents when adding new docs.

---
> Source: [matteopolak/portfolio](https://github.com/matteopolak/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
