## once-bigconfig

> This is the static website for Colors, an SDK for building Package Skills. The

# CLAUDE.md - AI Assistant Guide

## Project Overview

This is the static website for Colors, an SDK for building Package Skills. The
product was previously called BigConfig; the rebrand landed on 2026-07-27 and
reduced the site to **a single landing page**. The blog, the manual, the talk
decks, and the per-package pages were all removed at that point — do not
reintroduce them without being asked.

## Tech Stack

| Layer | Tool | Version |
|---|---|---|
| Site generator | Astro | ^6.3.1 |
| Image tooling | sharp | ^0.34.2 |
| Package manager | pnpm | v10.33.2 |
| TypeScript | strict Astro config | via `astro/tsconfigs/strict` |

That is the whole dependency list. MDX (`@astrojs/mdx`), D2 diagrams
(`astro-d2`), and the mdast/micromark packages were dependencies of the blog and
were removed with it. **Tailwind is also gone** — the landing page is styled with
inline attributes, so nothing imported it. If a future page wants Tailwind, add
`tailwindcss` + `@tailwindcss/vite` back, restore the vite plugin in
`astro.config.mjs`, and recreate `src/styles/global.css`.

## Repository Structure

```text
.
├── astro.config.mjs
├── Caddyfile.prod
├── package.json
├── pnpm-lock.yaml
├── plans/
│   └── colors-landing-page-export.html   # design reference, not built
├── public/
│   ├── fonts/            # 21 self-hosted IBM Plex woff2 files
│   ├── favicon.svg       # the three-stripe mark
│   └── og-colors.png     # og:image, generated — see scripts/
├── scripts/
│   └── generate-og-image.py
├── src/
│   ├── components/
│   │   └── SeoMeta.astro
│   └── pages/
│       ├── blog/
│       │   └── rss.xml.ts
│       └── index.astro
└── tsconfig.json
```

That is every source file. `public/` holds exactly three things — if you find
yourself adding a fourth, check it is actually referenced.

## Commands

```bash
pnpm dev
pnpm build
pnpm preview
pnpm astro
```

Always use `pnpm`.

## The landing page

`src/pages/index.astro` is the whole site. It was ported by hand from a bundled
design export, which is kept at `plans/colors-landing-page-export.html` as the
visual reference — that file is **not** built and must stay out of `src/pages/`
(it would publish as a route, and its filename contains spaces).

Things to know before editing it:

- The design is expressed as **inline `style` attributes**, not Tailwind classes.
  Match that when adding sections; don't mix idioms mid-page.
- The colour system is `oklch()` throughout. The three library accents are
  red `oklch(60% 0.19 25)`, green `oklch(65% 0.17 145)`, blue `oklch(55% 0.18 260)`.
- The three libraries are **red = TypeScript/Bun, green = Clojure/Babashka,
  blue = Python/uv**.
- The export shipped **no media queries**. The `@media` blocks at the end of the
  `<style is:global>` are additions, and they need `!important` to beat the
  inline styles. Classes `r-hero`, `r-2`, `r-3`, `r-4` mark the fixed grids.
- Hover states from the export became the `.h1` / `.h2` classes.
- The two "Copy" buttons are driven by `data-copy` and one small inline script
  at the bottom of the page. There is no framework — the page needs no JS to
  render, only to copy.
- Fonts are self-hosted from `public/fonts/`. Do not add a Google Fonts link.

## Brand assets

`public/favicon.svg` is the three-stripe mark: a 20×20 rounded square matching
the nav logo in `index.astro`. It uses hex rather than `oklch()` because not
every favicon rasteriser parses modern CSS colour — the oklch originals are in a
comment in the file.

`public/og-colors.png` is the og:image and is **generated**, not hand-drawn. Run
`scripts/generate-og-image.py` to rebuild it after any copy or brand change; the
setup block at the top of that script explains the one-off venv. It converts
text to outlines straight from `public/fonts/`, so the card's type matches the
page and no system fonts are needed, and it warns on stderr if a line overflows
the safe margin.

**Changing the artwork means changing the filename.** Slack, WhatsApp and
LinkedIn cache unfurls keyed on the image URL and hold them for days, so new
bytes at an old path keep showing the old card. Bump the name in both
`scripts/generate-og-image.py` (`OUT`) and `SeoMeta.astro` (`DEFAULT_IMAGE`).
The file was called `linkedin.png` under BigConfig; that name is retired.

## Testing a link unfurl before deploying

`og:image` and `canonical` are absolute URLs built from `site` in
`astro.config.mjs`, so a preview host serving a normal build still tells Slack
to fetch the image from production — and you see whatever production has today,
not your change. Build with the preview host instead:

```bash
SITE_URL=https://<preview-host> pnpm build
```

Production is unaffected: the Dockerfile runs a bare `pnpm build`, so `SITE_URL`
is unset and `site` falls back to the real domain. Do not commit a `dist/` built
this way.

Note the script synthesises the headline's semibold by stroking the outline —
`public/fonts/` only carries IBM Plex **Sans 400**. The page has the same gap and
lets the browser fake it. If you ever add a real Sans 600 woff2, drop the
`weight=` arguments and update the `@font-face` block in `index.astro`.

## RSS

`src/pages/blog/rss.xml.ts` survives the blog's deletion on purpose. It serves a
valid, **empty** RSS 2.0 channel at `/blog/rss.xml` so existing subscribers keep
the subscription rather than their reader dropping a 404 feed. It has no
dependencies and no items.

## Redirects

There is no `redirects` map in `astro.config.mjs` and no 404 page. Caddy handles
it: the `handle_errors` block in `Caddyfile.prod` 301s **any** 404 to `/`. That
covers every retired URL — the 44 blog posts, `/manual`, `/once`, `/walter`,
`/clickhouse`, `/marketplace`, `/package-spec`, the eight `/talk/*` decks, and
the old `/api/*` and `/libraries/*` docs routes — without enumerating them.

`/blog/rss.xml` is a real file, so `file_server` serves it and the redirect never
fires for it. `/up` is matched earlier and stays a 200 for the healthcheck.

**Gotcha, verified the hard way:** the matcher has to sit on `redir` itself.
Writing it as `handle @notFound { redir / 301 }` passes `caddy validate` and then
serves a bare 404 with no `Location` header. Do not "tidy" it back into a
`handle` block. After changing `Caddyfile.prod`, test it for real:

```bash
pnpm build
sed -e 's|^:80 {|:4321 {|' -e "s|root \* /srv|root * $PWD/dist|" \
  Caddyfile.prod > /tmp/Caddyfile.local
caddy run --config /tmp/Caddyfile.local --adapter caddyfile &
curl -sI localhost:4321/manual | head -2   # expect 301 + Location: /
```

## Notes

- `dist/` is generated by `pnpm build`.
- Do not edit `pnpm-lock.yaml` manually.
- Google Analytics ID: `G-4VKP1WY4QJ`.
- Site URL: `https://www.bigconfig.ai` — unchanged by the rebrand. The site is
  branded Colors but still served from the BigConfig domain.
- The install command on the page (`npx skills use bigconfig-ai/once`) and the
  three repo links (`github.com/amiorin/red|green|blue`) are shipped verbatim
  from the design export.

---
> Source: [bigconfig-ai/once-bigconfig](https://github.com/bigconfig-ai/once-bigconfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
