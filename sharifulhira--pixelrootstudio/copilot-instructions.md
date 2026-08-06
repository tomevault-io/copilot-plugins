## pixelrootstudio

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---

## Project: PixelRoot Studio — Professional Portfolio Website

**Founder:** Md Shariful Hira
**Brand principle:** Uncompromising Quality

### About the Photographer

Md Shariful Hira is a professional photographer with expertise across:
- **Fashion Photography**
- **Corporate Events**
- **Weddings**
- **Commercial Product Photography**
- **Cinematic Video Event Coverage**

His visual style is defined as **Modern Urban**: clean compositions, sharp details, elegant lighting, and sophisticated color tones. The overall aesthetic blends **cinematic storytelling** with **modern luxury** — timeless, polished, and emotionally powerful.

### Project Purpose

This is PixelRoot Studio's **SEO-first marketing and portfolio website**. The site targets brands, businesses, and individuals seeking premium-quality visual storytelling.

### Design Direction

- Luxury, high-end aesthetic
- Modern urban visual style — clean, sharp, elegant, sophisticated
- Cinematic color palette (deep blacks, warm ambers/gold, clean whites)
- Mobile-first, performance-optimized
- Full-bleed sections — pages own their layout; avoid wrapping in `.content-wrap` for full-bleed pages

### Tech Stack

- **Framework:** Next.js 16 (App Router, `force-static` homepage)
- **Language:** TypeScript
- **Styling:** Tailwind CSS v4 (no config file — uses CSS-first config in `globals.css`)
- **Animation:** Framer Motion v12
- **Fonts:** Inter (body) + Playfair Display (headlines) via `next/font/google`, exposed as CSS variables `--font-inter` / `--font-playfair`
- **SEO:** JSON-LD structured data, Open Graph, sitemap, robots.txt
- **Data:** JSON files in `data/` folder — components import them directly (no fetch)

### File Conventions

- Page sections live in `components/home/`
- Shared data lives in `data/*.json`
- SEO utilities live in `seo/`
- The homepage (`app/page.tsx`) is `force-static` — no server-side dynamic calls
- Client components that need hooks/animations use `"use client"`
- Server components import JSON data directly via `import data from "@/data/file.json"`

### CSS Variables (globals.css)

Light mode defaults, dark mode via `html[data-theme="dark"]`:
- `--bg`, `--surface`, `--text`, `--muted`, `--border`
- `--accent` / `--primary`: brand blue (`#0b3a75` light / `#4a83d8` dark)
- Hero sections always use dark cinematic style (black overlay on photo backgrounds), independent of theme

### Accent Colors

- **Brand blue:** `var(--primary)` — used for UI chrome, nav, buttons
- **Gold/amber:** `#d4a843` / Tailwind `amber-400` — used as luxury accent on hero/showcase sections

---
> Source: [sharifulhira/PixelRootStudio](https://github.com/sharifulhira/PixelRootStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
