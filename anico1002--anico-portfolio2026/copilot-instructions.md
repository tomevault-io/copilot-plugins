## anico-portfolio2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Dev server on localhost:3000
npm run dev:mobile   # Dev server on 0.0.0.0 (for testing on real device)
npm run build        # Production build
npm run lint         # ESLint
```

After build/content changes: delete `.next/` and restart the server to pick up changes to `public/` assets.

## Architecture

**Stack:** Next.js 16 + React 19 + TypeScript + Tailwind CSS v4 + framer-motion.

**Routes:**
- `/` — Home (Nav + Hero carousel + Marquee + Work grid + About + Contact + Footer)
- `/proyecto/[slug]` — Project detail (static pre-rendered via `generateStaticParams`)

**Data flow — two sources merged at request time:**

1. **`lib/projects.ts`** — Static array of `Project` objects (baseline metadata + CDN image URLs)
2. **`public/projects/<slug>/`** — File-based CMS that overrides/enriches the static data:
   - `content.txt` → metadata, text blocks, stats, links (parsed by `lib/parse-content.ts`)
   - `*_hero.(png|jpg|…)` → project hero image
   - `*_thumb.(png|jpg|…)` → thumbnail
   - `NN_Section-Name/` subfolders → visual sections with titles
   - Image groups: `1.png` = single image; `1a.png + 1b.png` = fade slideshow
   - `*_hero.(mp4|webm|mov)` → project hero **video** (preferred over static image — renders as `<video autoPlay loop muted playsInline>`, no pixelation)
   - `*_phone.(mp4|jpg|png|…)` → media inside the CSS phone frame
   - `*_phone_bg.(png|jpg|…)` → background image behind the phone mockup

`lib/scan-project.ts` → `getEnrichedProjects()` merges both sources and auto-discovers new folders. Project pages use `dynamic = "force-dynamic"` so content.txt changes are visible on reload.

**GIF/WebP → MP4 conversion** (no ImageMagick; ffmpeg via npm global):
```bash
FFMPEG=/Users/anico/.npm-global/lib/node_modules/ffmpeg-static/ffmpeg
# Animated GIF or static input → MP4
$FFMPEG -i input.gif -vcodec libx264 -crf 22 -preset slow \
  -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" -pix_fmt yuv420p -an -movflags +faststart -y out.mp4
# Animated WebP (ffmpeg can't decode ANIM chunks directly — extract frames first):
python3 -c "
from PIL import Image; import os
img=Image.open('in.webp'); os.makedirs('/tmp/fr',exist_ok=True)
[img.seek(i) or img.convert('RGBA').save(f'/tmp/fr/{i:04d}.png') for i in range(img.n_frames)]
"
$FFMPEG -framerate 25 -i /tmp/fr/%04d.png -vcodec libx264 -crf 22 -preset slow \
  -pix_fmt yuv420p -an -movflags +faststart -y out.mp4
```

**Hero carousel:** `public/hero-imgs/` — files sorted alphabetically, support images (jpg/png/gif/webp/avif) and videos (mp4/webm/mov). Videos render as `<video autoPlay loop muted playsInline>`.

## Content editing

| What | Where |
|------|-------|
| Home/About/Contact copy | `lib/i18n.ts` |
| Site-wide copy (hero, about, marquee) | `public/profile.txt` |
| Project text (overview, challenge, etc.) | `public/projects/<slug>/content.txt` |
| Hero carousel | Add/remove files in `public/hero-imgs/` |
| Disable a project | Add slug to `DISABLED_PROJECT_SLUGS` in `lib/scan-project.ts` |

**content.txt metadata keys:** `name`, `subtitle`, `client`, `category`, `year`, `role`, `deliverables`, `tags`, `phonecolor` (black|white|starlight), `phoneratio` (e.g. 9/16 or 393/852), `phonemodel` (iphone8|iphone14 — default: iphone8)

**content.txt blocks:** `[overview]`, `[challenge]`, `[process]`, `[outcome]`, `[stats]` (one line = one stat box), `[links]` (Label | URL), `[button]` (Label https://...)

**Phone mockup** (`components/PhoneMockup.tsx`): renders a CSS phone frame around a video or image. Two models supported:
- **iPhone 8** (default): natural size 423×877px (content-box), CSS from marvelapp/devices.css. Screen stroke `#bfbfc0` white / `#3c3d3d` black.
- **iPhone 14**: natural size 428×868px (border-box, screen 390×830), CSS from picturepan2/devices.css. `white`/`starlight` → `.device-starlight` class; black = no extra class.

Both use `.ph-scaler` + `.ph-wrap` with `--ps` CSS variable for responsive scaling (0.33→0.63). Background uses `aspect-[12/5]` (1920×800) so bg image shows without cropping.

**HEVC/H.265 → H.264 for phone video** (source files are often iPhone screen recordings):
```bash
FFMPEG=/Users/anico/.npm-global/lib/node_modules/ffmpeg-static/ffmpeg
$FFMPEG -i input_src.mp4 -vcodec libx264 -crf 26 -preset slow \
  -vf "scale=392:-2" -r 30 -pix_fmt yuv420p -an -movflags +faststart -y output_phone.mp4
```
- 392px width (even number, matches iPhone 14 ratio 393×852)
- Remove source file after conversion — never commit `*_src.mp4` files

## UI conventions

- **Content width:** `max-w-6xl mx-auto` with `px-6 md:px-12`
- **Images:** always `object-cover`, never `object-contain`; use `aspect-[4/3]` or `aspect-[16/10]` ratios (no fixed px heights)
- **Mobile project images:** full-width with `w-screen relative left-1/2 -translate-x-1/2` wrapper; `md:relative md:left-0 md:translate-x-0 md:w-full`
- **Mobile full-bleed within padded section:** use `-mx-6 md:mx-0` to break out of `px-6`; add `-mt-24 md:mt-0` to also remove top section padding
- **Typography weight:** headlines use `font-display font-extrabold`; `font-semibold` is not used for display text
- **Hero zoom on mobile:** parallax (`imageY`) is disabled on mobile via `isMobile` state + `window.matchMedia`; zoom CSS animation needs `willChange: 'transform'` for iOS Safari GPU compositing
- **Logo:** `public/anico_logo.svg` at `h-[22px] w-auto`; add `invert` on dark backgrounds
- **Theme tokens** (Tailwind v4 inline in `globals.css`): `background` #fff, `foreground`/`primary` #0a0a0a, `muted` #f5f5f5, `border` #e5e5e5
- **Locale:** hardcoded `"en"` in `context/AppContext.tsx` — the Spanish translations in `lib/i18n.ts` exist but are unused

## SEO & GEO

- **`app/sitemap.ts`** — auto-generates `/sitemap.xml` from `getEnrichedProjects()`
- **`app/robots.ts`** — `/robots.txt` with AI crawler permissions (GPTBot, PerplexityBot, ClaudeBot, anthropic-ai)
- **`components/JsonLd.tsx`** — exports `<PersonSchema>` (injected in `layout.tsx` `<head>`) and `<CreativeWorkSchema>` (injected in each project page)
- **`public/llms.txt`** — plain-text site description for LLM crawlers (GEO standard)
- **OG image:** uses `/about.webp` provisionally — replace with a proper 1200×630px `/og-image.png` for ideal social sharing

## Unused components

- `components/Skills.tsx` — exists but is **not mounted** in `app/page.tsx`
- `components/CustomCursor.tsx` — exists but is **not used** (system cursor active; no `cursor-none` on body)

## Deployment

Vercel project: `anico_portfolio2026` → domain `anico.design`. Push to `main` triggers auto-deploy.

---
> Source: [anico1002/anico_portfolio2026](https://github.com/anico1002/anico_portfolio2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
