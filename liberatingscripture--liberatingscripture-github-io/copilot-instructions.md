## liberatingscripture-github-io

> This is the **Liberating Scripture Collective** (LSC) website. LSC is the nonprofit

# liberatingscripture.org — Claude Code Instructions

## Project Overview

This is the **Liberating Scripture Collective** (LSC) website. LSC is the nonprofit
organization behind the LIT Bible. The translation product lives at litbible.net;
this site is the organizational home.

## Tech Stack

- **Framework**: Astro 5 (static site generator)
- **Language**: TypeScript (strict mode)
- **Styling**: Vanilla CSS — no utility framework
- **Fonts**: Self-hosted via @fontsource (Crimson Text, Inter, Fraunces)
- **Deploy**: GitHub Pages → liberatingscripture.org (CNAME in public/)

## Commands

```bash
npm install         # Install dependencies
npm run dev         # Dev server at localhost:4321
npm run build       # Production build to dist/
npm run preview     # Preview the build
```

## Structure

```
src/
  components/
    SiteHeader.astro    # Sticky header with mobile menu
    SiteFooter.astro    # Dark footer, 4-column layout
  layouts/
    Layout.astro        # Base HTML shell (SEO, fonts, header/footer)
  pages/
    index.astro         # Homepage
    about.astro         # About LSC
    lit-bible.astro     # Internal landing page for the LIT Bible
    support.astro       # Donate + get involved (Give Lively placeholder)
    podcasts.astro      # Hub for both podcasts
    community.astro     # Community & Courses (5th project)
    spiritual-direction.astro
    contact.astro
    404.astro
  styles/
    global.css          # Full design system — do not change token values
public/
  assets/
    images/             # ← Image assets go here (see below)
    og/                 # ← OG images go here
  CNAME
  robots.txt
  site.webmanifest
  favicon*.png etc.
```

## Image Assets Needed

Drop these files into `public/assets/images/`:

- `lsc-logo.png` — main LSC logo (used in header, footer, homepage)
- `fit-cover.webp` — Found in Translation podcast cover art
- `twb-banner.png` — The Table We're Building banner/cover

Drop these into `public/assets/og/`:
- `og-default.png` — 1200×630 OG image
- `og-square.png` — 1200×1200 OG image

Drop these into `public/` (root):
- `favicon.ico`
- `favicon-16x16.png`
- `favicon-32x32.png`
- `apple-touch-icon.png`
- `android-chrome-192x192.png`
- `android-chrome-512x512.png`

## Design System

Colors (do not change):
- `--cream: #E1DFD9` — page background
- `--green: #209D50` — brand primary
- `--ink: #1D231C` — text / dark CTA bg
- `--white: #FFFFFF` — surface raised
- `--black: #000000` — strong text

Fonts: Crimson Text (headings) · Inter (body) · Fraunces (display text)

## Things to Replace Before Launch

1. ~~**Give Lively embed**~~ — live as of 2026-05-11. Widget embedded in `src/pages/support.astro`.

2. ~~**501(c)(3) language**~~ — updated 2026-05-11 in both `support.astro` and `about.astro`; status is active.

3. **Formspree ID** — the contact form uses `https://formspree.io/f/xdkqvlkj`.
   Update in `src/pages/contact.astro` if the ID changes.

4. **Cloudflare Turnstile sitekey** — `0x4AAAAAACJ446flkL7Rwf8i` in
   `src/pages/contact.astro`. Update if it changes.

5. **Working values** — the five placeholder values in `src/pages/index.astro`
   (in the `values` array) should be replaced with the finalized LSC values
   statement once drafted.

## Key Relationships

- litbible.net is the LIT Bible product site (Astro 5, same design system)
- liberatingscripture.org is the org site (this repo)
- litbible.net/liberating-scripture-collective links here for more info
- This site links to litbible.net for the translation, FIT podcast, and LSC page

## Existing External Integrations

- **Formspree**: contact form submissions → email
- **Cloudflare Turnstile**: bot protection on contact form
- **Give Lively**: donation platform (live — slug: liberating-scripture-collective)
- **Apple Podcasts**: FIT podcast ID 1586737797
- **Spotify**: FIT podcast ID 6S2wWaM5oqknwncPfOEyZ6
- **YouTube**: @foundintranslationpodcast

---
> Source: [liberatingscripture/liberatingscripture.github.io](https://github.com/liberatingscripture/liberatingscripture.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
