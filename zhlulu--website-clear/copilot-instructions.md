## website-clear

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Public website for the **CLEAR Center** — *Center for All-Clear SEP Forecast* — a NASA Space Weather Center of Excellence led by PI Lulu Zhao (University of Michigan, CLaSP), building an integrated prediction framework for Solar Energetic Particles (SEPs). The site is informational/static; there is no auth, no backend, no CMS.

## Commands

```bash
npm install         # first-time setup
npm run dev         # local dev server at http://localhost:4321
npm run build       # static build → dist/
npm run preview     # serve the built site locally
```

There is no test suite. The build itself is the primary verification: `npm run build` will fail on type errors, broken Markdown frontmatter, or broken content-collection schemas.

## Architecture

Astro 4 + Tailwind 3, fully static output.

- **`src/layouts/BaseLayout.astro`** — the single site shell (head, `<Header>`, `<Footer>`, skip link). Every page renders inside it; don't duplicate head/nav markup elsewhere.
- **`src/components/Section.astro`** — standard section wrapper with an optional `eyebrow` / `title` / `lede` header block. Use it for new content sections to keep spacing and typography consistent across pages.
- **`src/pages/news/index.astro`** — "In the news" is an inline `items` array of external article links (`title`, `date`, `source`, `url`, `summary`), sorted by date at render time. Adding coverage = adding an array entry; some entries are kept commented out until cleared for publication.
- **`src/pages/sofie.astro`** — per-tool detail page. Tool cards in `src/pages/tools.astro` can point to one via their `learnMore` field; use SOFIE as the template when adding detail pages for other tools (SEPNET, AWSoM, …).
- **`_import/`** — static HTML snapshots of the old CLEAR site (home/science/team/tools). Reference material only; not part of the build.
- **`tailwind.config.mjs`** — University of Michigan brand palette: `maize` (official UM Maize #FFCB05, Pantone 7406) and `umblue` (official UM Blue #00274C, Pantone 282), each as a 50–900 scale. Stick to these tokens rather than raw Tailwind colors (`yellow-*`, `blue-*`) so the palette can be retuned centrally.
- **`src/styles/global.css`** — defines reusable classes (`container-narrow`, `container-wide`, `btn-primary`, `btn-maize`, `btn-secondary`, `eyebrow`, `.starfield`). Prefer these over re-declaring the same utility strings on every page.

### Color use conventions

- Body text & headings on white: `text-umblue-800` or `text-umblue-900`. Maize on white has poor contrast — don't use `text-maize-600/700` for readable body/heading text, only as decorative borders, bullets, badges on tinted backgrounds, or text on dark UM-blue backgrounds.
- `.btn-primary` = solid UM Blue button (default CTA). `.btn-maize` = UM Maize button with UM Blue text, reserved for highlighted CTAs (e.g., hero, accent cards).
- On dark UM-blue backgrounds, accent text should be `text-maize-300/400` (high contrast, iconic UM).

## Content conventions

- The proposal PDF that seeded this site lives at `/Users/zhlulu/University of Michigan Dropbox/Lulu Zhao/03_PROPOSALS/funded/2022/Zhao_SWxC/CLEAR_proposal.pdf`. Treat it as the source of truth for scientific claims — don't invent figures (award amount, forecast horizon, institution list, module names).
- Team data lives in inline arrays in `src/pages/team.astro` (`executive`, `advisoryBoard`, `operationalGroups`, `memberGroups`); headshots are under `public/team/`. If it grows or changes often, move it to a JSON/TS data file under `src/data/` — don't scatter duplicates across pages.
- The `clear.png` logo in `public/` is the official center logo (copied from the proposal folder). The favicon is `favicon.png` — the logo's figure mark recolored solid maize, on a transparent background (text is illegible at 16px; the original blue lettering vanished on dark tab bars). Bump the `?v=` query on the icon link in `BaseLayout.astro` whenever the favicon changes, or browsers keep the cached one. `favicon.svg` is a leftover stylized sun no longer referenced.

## Deployment

Live at **https://clear.engin.umich.edu** — GitHub Pages behind a UM-managed CNAME (`clear.engin.umich.edu` → `zhlulu.github.io`). Every push to `main` auto-builds and deploys via `.github/workflows/deploy.yml` (withastro/action), so **pushing to `main` publishes the site** — only push when the user asks. The custom domain requires `public/CNAME` and the `site` value in `astro.config.mjs`; that file's comment shows how to revert to the plain GitHub Pages URL (restore `base: "/Website_CLEAR/"`). Pages still build links with `import.meta.env.BASE_URL` so a base path would keep working.

---
> Source: [zhlulu/Website_CLEAR](https://github.com/zhlulu/Website_CLEAR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
