## jordanhilario

> Personal website for Jordan Hilario, a fingerstyle guitarist. The site drives traffic to the YouTube channel (@JordanHilarioFingerstyle) and sells guitar tabs via MyMusic5. Built as a static site with Astro 5, deployed on Vercel.

# jordanhilario.com — Fingerstyle Guitar Website

## Project Overview
Personal website for Jordan Hilario, a fingerstyle guitarist. The site drives traffic to the YouTube channel (@JordanHilarioFingerstyle) and sells guitar tabs via MyMusic5. Built as a static site with Astro 5, deployed on Vercel.

## Tech Stack
- **Framework:** Astro 5 (static output, zero JS by default)
- **Hosting:** Vercel (free tier, auto-deploys from GitHub on push)
- **Repository:** https://github.com/flytying/jordanhilario (public)
- **Domain:** jordanhilario.com (registered on DreamHost, nameservers pointed to Vercel)
- **Contact form:** Formspree (endpoint: `mlgojpvb`)
- **Analytics:** Google Analytics GA4 (`G-XF3H4BXFKL`)
- **Ads:** Google AdSense (`ca-pub-3392449098044156`)
- **Tab sales:** https://www.mymusic5.com/jordanhilariofingerstyle

## Site Architecture
```
/ (Home)                          — Hero with featured video, About section, Featured covers grid, CTA banner
/fingerstyle-videos               — All videos grid with clickable tag filters
/fingerstyle-videos/[slug]        — Individual video page with YouTube embed, tab purchase CTA, related videos
/contact                          — Contact form (Formspree) + social links
```

## Key Directories
```
src/
├── components/       Header, Footer, VideoCard, VideoPlayer, SEOHead
├── content/videos/   One .md file per video (31 total)
├── layouts/          BaseLayout.astro (wraps all pages)
├── pages/            index, fingerstyle-videos/, contact
├── styles/           global.css (CSS custom properties, no framework)
├── utils/            youtube.ts (thumbnail URL helper)
└── content.config.ts Astro content collection schema
```

## Content Model (Video)
Each video is a markdown file in `src/content/videos/`. Frontmatter schema:
```yaml
title: "Song Name - Artist Fingerstyle Guitar Cover"
songTitle: "Song Name"
artist: "Artist Name"
youtubeId: "YouTube_VIDEO_ID"
publishDate: 2025-01-15
description: "SEO description for meta tags"
tags: ["opm", "ballad"]           # Used for filtering on /fingerstyle-videos
tabUrl: "https://mymusic5.com/jordanhilariofingerstyle/12345"  # Optional — shows "Tab Available" badge + purchase CTA
featured: false                    # true = appears in homepage Featured Covers grid
```

## Adding a New Video
1. Create `src/content/videos/song-name-slug.md` with frontmatter above
2. Set `featured: true` if it should appear on the homepage (currently 6 featured)
3. Add `tabUrl` only if a tab exists on MyMusic5
4. `git add . && git commit -m "Add: Song Name" && git push`
5. Vercel auto-deploys in ~30 seconds

## Hero Video
The hero video on the homepage is the **most recent** video with `featured: true` (sorted by `publishDate`). Currently: "Let You Break My Heart Again" by Laufey (`publishDate: 2025-03-20`). To change the hero, either update the date or swap `featured` flags.

## Featured Videos (Homepage)
Currently 6 featured videos displayed in a 3x2 grid:
- Let You Break My Heart Again (Laufey)
- Dance With Me (Orleans)
- Somebody (Depeche Mode)
- Here Comes The Sun (The Beatles)
- Love is Real (John Lennon)
- If You Leave Me Now (Chicago)

## SEO
- **URL structure:** `/fingerstyle-videos/song-slug` (keyword "fingerstyle" in URL)
- **Homepage title:** "Fingerstyle Guitar Cover & Tabs | Jordan Hilario"
- **Structured data:** WebSite + Person on home, CollectionPage on listing, VideoObject on each video page
- **Sitemap:** Auto-generated at `/sitemap-index.xml` via @astrojs/sitemap
- **robots.txt:** In `public/robots.txt`
- **Google Search Console:** Verified via Vercel DNS TXT record

## Design System
- **Style:** Bold & editorial — large uppercase headings, clean sans-serif body
- **Colors:** Black (`#0a0a0a`), White (`#ffffff`), Red accent (`#e63946`), Muted gray (`#6b7280`)
- **Font:** Inter / system font stack
- **Layout:** Dark header/footer, white content areas, CSS Grid for video cards
- **Mobile:** Responsive with hamburger nav (CSS-only `<details>`), full-width buttons

## Performance
- YouTube thumbnails from CDN (`mqdefault` 320x180) — no hosted images
- `lite-youtube-embed` — only loads YouTube iframe on click
- CSS inlined via `build.inlineStylesheets: 'always'`
- GA + AdSense scripts at end of `<body>` to avoid render-blocking
- CSS containment on video player to prevent layout shift

## Security Headers (vercel.json)
Content-Security-Policy, Strict-Transport-Security (HSTS with preload), Cross-Origin-Opener-Policy, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy

## Domain / DNS
- Registered on **DreamHost**
- Nameservers pointed to **Vercel** (not using DreamHost DNS)
- No domain email (separate paid add-on on DreamHost)

## Third-Party Services
| Service | Purpose | Config Location |
|---------|---------|-----------------|
| Formspree | Contact form submissions | `src/pages/contact.astro` (endpoint ID) |
| Google Analytics | Traffic tracking | `src/layouts/BaseLayout.astro` |
| Google AdSense | Ads | `src/layouts/BaseLayout.astro` |
| YouTube CDN | Video thumbnails + embeds | `src/utils/youtube.ts`, `src/components/VideoPlayer.astro` |
| MyMusic5 | Guitar tab sales | `tabUrl` in video frontmatter + CTA links throughout |

## Commands
```bash
npm run dev      # Start dev server on port 4321
npm run build    # Build static site to dist/
npm run preview  # Preview production build locally
```

## Notes
- Jordan does not record videos on a regular schedule — avoid "every week" language
- Content spans OPM classics, 80s ballads, Beatles, Laufey, jazz standards, classic rock
- The logo file is at `public/jordanhilario-fingerstyleguitar.png` but the header/footer currently use text "JORDAN HILARIO"
- 22 of 31 videos have individual tab purchase URLs on MyMusic5
- Tag filtering on `/fingerstyle-videos` uses client-side JS (only JS on the site besides lite-youtube)

---
> Source: [flytying/jordanhilario](https://github.com/flytying/jordanhilario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
