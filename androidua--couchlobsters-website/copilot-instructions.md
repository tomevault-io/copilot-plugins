## couchlobsters-website

> This file tells Claude Code everything it needs to know about this project.

# Couch Lobsters Website — Project Context for Claude Code

This file tells Claude Code everything it needs to know about this project.
Place this file in the root of the `couchlobsters-website` repo.

---

## What This Project Is

A static podcast website for **Couch Lobsters** — a film & TV series podcast hosted by
Jess & Dima. Built as plain HTML/CSS/JS (no frameworks, no build step required).

**Live domain:** couchlobsters.com  
**GitHub repo:** https://github.com/androidua/couchlobsters-website  
**Hosting:** Cloudflare Pages (auto-deploys when changes are pushed to `main` branch)  
**Workflow:** Edit files locally → `git push` → Cloudflare auto-deploys within ~2 minutes

---

## File Structure

```
couchlobsters-website/
├── index.html          ← Homepage (hero, concept, latest episodes, platform links, upcoming episode teasers)
├── episodes.html       ← All episodes page (full grid of all episodes)
├── watching.html       ← What We're Watching page (filterable grid: status × person × year)
├── about.html          ← About page (show description + host bios)
├── style.css           ← All styles (dark cinematic theme, gold accent #e8c96d)
├── episodes-data.js    ← Episode data array + UPCOMING_EPISODES teaser config (data only — no functions)
├── watching-data.js    ← Watching picks — auto-generated from Google Sheets (do not edit manually)
├── main.js             ← Nav toggle + episode/teaser card rendering + watching page logic + all helpers
├── logo.jpg            ← Self-hosted podcast logo (640×640 JPEG) — nav/hero/footer images + og:image on all pages
├── favicon.jpg         ← Self-hosted podcast logo (300×300 JPEG) — used as <link rel="icon"> on all pages
├── favicon.ico         ← ICO binary (16×16 + 64×64 PNG) — required for Safari's /favicon.ico domain lookup
├── robots.txt          ← Allow all + sitemap pointer
├── sitemap.xml         ← All 4 pages with <lastmod> (episode sync auto-refreshes / and /episodes.html)
├── _headers            ← Cloudflare Pages security headers (CSP, X-Frame-Options, etc.)
├── .github/workflows/sync-watching.yml  ← 6-hourly GitHub Action: Google Sheets CSV → watching-data.js
├── .github/workflows/sync-episodes.yml  ← Daily GitHub Action: RSS → episodes-data.js + follow-up issue
├── .github/workflows/validate.yml       ← CI guardrail: validates data files + HTML invariants on every push
├── .github/scripts/sync_episodes.py     ← RSS parser (fills Apple links via iTunes API, updates sitemap)
├── .github/scripts/validate.js          ← Validation script shared by CI and both sync workflows
└── CLAUDE.md           ← This file
```

---

## How the Site Works

- **No build tools.** Pure HTML, CSS, JavaScript. No npm, no webpack, nothing to install.
- **Episodes are data-driven.** All episode info lives in `episodes-data.js` as a JS array called `EPISODES`.
  Both `index.html` (shows latest 6 on desktop, 4 on mobile) and `episodes.html` (shows all) pull from this same array.
- **Upcoming-episode teasers** are driven by the `UPCOMING_EPISODES` array in `episodes-data.js` (one card per
  entry; `status: "recorded"` shows a "✳ Recorded · Releasing Soon" badge). Set to `[]` to hide the section.
- **Episodes auto-sync daily.** `.github/workflows/sync-episodes.yml` fetches the RSS feed, prepends new episodes
  to `EPISODES`, fills per-episode Apple links via the iTunes Lookup API, refreshes sitemap `lastmod`, and opens
  a GitHub issue listing the remaining manual steps (Spotify per-episode link, teaser update). Spotify episode
  URLs are NOT available via any unauthenticated API — new episodes get the show URL until pasted manually.
- **Episode artwork** is hotlinked directly from the podcast RSS feed CDN (podcloud.fr).
- **What We're Watching** is a filterable page (`watching.html`) showing picks by Jess & Dima.
  Data lives in `watching-data.js` (auto-generated — do not edit manually).
  Managed via Google Sheets; a GitHub Actions workflow (`.github/workflows/sync-watching.yml`) fetches the sheet
  as CSV every 6 hours and commits `watching-data.js` only when content has changed.
  Uses `var WATCHING` (not `const`) — Safari scopes top-level `const` to the declaring script; `var` attaches
  to `window` and is visible across all script tags. Do not change to `const`.
- **CI validation** (`.github/workflows/validate.yml` → `.github/scripts/validate.js`) runs on every push and
  inside both sync workflows before committing: JS syntax, episode/watching data shape, https-only URLs,
  no bare `rel="noopener"`, no inline `onerror=`, no hardcoded episode counts, sitemap well-formedness.
- **Brand images are self-hosted**: `logo.jpg` (640×640, nav/hero/footer + og:image), `favicon.jpg` (300×300,
  `<link rel="icon">` on all pages) + `favicon.ico` (binary ICO in repo root — Safari auto-requests
  `/favicon.ico` for root-domain URLs; without this file Cloudflare Pages serves `index.html` instead, causing
  the tab icon to fall back to a letter). Never hotlink brand images from the Spotify CDN — those URLs rotate.
- **Fonts** are loaded from Google Fonts: Bebas Neue (display), DM Sans (body), Playfair Display (italic accents).
- **Security headers** are set in `_headers` (Cloudflare Pages format). CSP allowlists Cloudflare Insights, Google Fonts, and podcast image CDNs.

---

## Design System

| Element | Value |
|---------|-------|
| Background | `#0d0d0d` (near black) |
| Card background | `#1c1c1c` |
| Accent colour | `#e8c96d` (warm gold) |
| Danger/warning | `#c94b3a` (deep red) |
| Text | `#f0ece4` |
| Muted text | `#888` |
| Display font | Bebas Neue |
| Body font | DM Sans |
| Italic accent font | Playfair Display |

---

## About the Podcast

- **Name:** Couch Lobsters
- **Hosts:** Jess (Jessica Schaltin) & Dima (Dmytro)
- **Format:** Each episode, they assign each other a film or TV series to watch.
  Opinions are kept secret until recording day. Full spoilers throughout.
- **Tagline:** "The film & series podcast made by amateurs for cinema enthusiasts."
- **Episodes:** 27 published (as of July 2026), released roughly every 4–8 weeks — live count is always `EPISODES.length`
- **Based in:** Australia

### Platform Links
- Spotify: https://open.spotify.com/show/6KbzgmH3YRS2mc0cbjd82y
- Apple Podcasts: https://podcasts.apple.com/au/podcast/couch-lobsters/id1681472927
- Deezer: https://www.deezer.com/en/show/5945017
- Instagram: https://www.instagram.com/couchlobsters/
- Facebook: https://facebook.com/couchlobsters
- RSS Feed: https://couch-lobsters.lepodcast.fr/rss

### Host Social Links
- Dima Instagram: https://www.instagram.com/androidua/
- Dima Facebook: https://www.facebook.com/dima.bond
- Jess Instagram: https://www.instagram.com/jessschltn/
- Jess Facebook: https://www.facebook.com/jessica.schaltin
- Jess Bluesky: @Bad_Penguin

---

## How to Update the Upcoming Episode Teasers

The homepage "Next Episode" section renders one card per entry of the `UPCOMING_EPISODES`
array at the top of `episodes-data.js` (two cards sit side by side on desktop).

**The user only needs to say:** the two film names (with years), plus whether the episode is
already recorded or still to be recorded (and an expected/recording date if known).
Claude should handle everything else — finding artwork URLs from TMDB and updating the file.

```javascript
const UPCOMING_EPISODES = [
  {
    films: ["Film A (year)", "Film B (year)"],   // shown as "A VS B" on the card
    artworks: [
      "https://...",   // poster for Film A — use TMDB: https://www.themoviedb.org/
      "https://..."    // poster for Film B — TMDB URL format: https://media.themoviedb.org/t/p/w500/POSTER_PATH.jpg
    ],
    status: "recorded",       // "recorded" → "✳ Recorded · Releasing Soon" badge
                              // "scheduled" → "Coming Soon" badge (+ expectedDate if set)
    label: "Next Episode",    // optional card label; defaults to "Next Episode"
    teaser: null,             // optional one-line tagline e.g. "Two classics. One winner." — or null
    expectedDate: null        // free-form string e.g. "Recording 11 July" — or null to omit
  }
];
```

**When an episode is published:** remove its entry from `UPCOMING_EPISODES` (the daily
episode sync adds it to `EPISODES` automatically, and the sync's follow-up issue reminds you).

**To hide the section entirely** (e.g. between seasons): set `UPCOMING_EPISODES = []`.

**TMDB poster URL pattern:** `https://media.themoviedb.org/t/p/w500/POSTER_PATH.jpg`
Find it by searching the film on https://www.themoviedb.org/ and copying the poster path from the image URL.

---

## How to Add a New Episode

**Normally automatic.** The daily `sync-episodes.yml` workflow adds new episodes from the
RSS feed (including Apple links and sitemap updates) and opens a follow-up issue for the
two remaining manual steps: the Spotify per-episode link and the `UPCOMING_EPISODES` cleanup.

To add one manually (or fix a synced entry), edit the **top** of the `EPISODES` array
in `episodes-data.js`. Each episode object looks like this:

```javascript
{
  num: 28,                          // Episode number
  title: "Film A (year) VS Film B (year)",
  date: "2026-03-15",               // YYYY-MM-DD format
  duration: "1h 45m",
  artwork: "https://...",           // URL from RSS feed itunes:image tag
  spotifyUrl: "https://open.spotify.com/episode/...",
  appleUrl: "https://podcasts.apple.com/au/podcast/...",
  films: ["Film A (year)", "Film B (year)"]
}
```

To find the artwork URL and episode links for a new episode, check the RSS feed:
https://couch-lobsters.lepodcast.fr/rss

---

## Deploying Changes

This is a **solo project** — push directly to `main`. No branches, no pull requests.

```bash
git add .
git commit -m "Brief description of what changed"
git push
```

Cloudflare Pages will auto-deploy within ~2 minutes.
The live site is at https://couchlobsters.com.

---

## Versioning

Semantic versioning — bump in README badge + git tag in same commit:
- **Patch** (x.x.N): bug fixes, responsive tweaks, copy changes
- **Minor** (x.N.0): new features (e.g. teaser section, episode sync)
- **Major** (N.0.0): significant redesigns

```bash
git tag vX.Y.Z && git push origin vX.Y.Z
```

Current version: **v1.4.0**

### Pre-commit README checklist (minor and major bumps)

Before committing a **minor or major** version bump, always check:
- [ ] README version badge updated (`![Version](https://img.shields.io/badge/version-X.Y.Z-e8c96d)`)
- [ ] README **Tech Stack** still accurately describes how the site works
- [ ] README **Project Structure** lists all key files
- [ ] CLAUDE.md **File Structure** and **How the Site Works** are up to date

---

## Things Still To Do

- [ ] Consider adding individual episode pages (optional — not planned yet)
- [ ] Add host photos to About page when available
- [ ] Paste per-episode Spotify links for the 14 older episodes still using the show URL
      (not fetchable without authenticated Spotify API access; list in the v1.4.0 commit message)

---

## SEO Standards

Every page must maintain all of the following. Never add a page without them.

### Required per-page tags
- `<title>` — unique, descriptive, under ~60 characters
- `<meta name="description">` — unique, 140–160 characters
- `<link rel="canonical">` — full absolute URL
- `<link rel="alternate" type="application/rss+xml">` — podcast RSS autodiscovery

### Open Graph (all required)
```html
<meta property="og:type" content="website">
<meta property="og:site_name" content="Couch Lobsters">
<meta property="og:locale" content="en_AU">
<meta property="og:url" content="https://couchlobsters.com/PAGE">
<meta property="og:title" content="…">
<meta property="og:description" content="…">
<meta property="og:image" content="…">
<meta property="og:image:width" content="640">
<meta property="og:image:height" content="640">
<meta property="og:image:alt" content="…">
```

### Twitter Card (all required)
```html
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="…">
<meta name="twitter:description" content="…">
<meta name="twitter:image" content="…">
```
`summary` (not `summary_large_image`) — the share image is the square 640×640 logo;
the large card format expects 2:1 landscape art and would crop it badly.

### JSON-LD structured data
- `index.html`: `PodcastSeries` + `WebSite` in a `@graph`
- `episodes.html`: `BreadcrumbList` (static) + `ItemList` of `PodcastEpisode` (injected by `main.js`)
- `watching.html` / `about.html`: `BreadcrumbList` (+ `Person` × 2 on about)
- `<script type="application/ld+json">` is CSP-exempt — no `script-src` changes needed

### sitemap.xml
All public pages must be listed with `<lastmod>` dates. Update `lastmod` whenever content on a page changes significantly. `watching.html` uses `changefreq="weekly"` (auto-synced from Sheets).

### Links
All external links must use `target="_blank" rel="noopener noreferrer"` — both attributes are required.

---

## Security Standards

Security headers live in `_headers` (Cloudflare Pages format). Keep all of these present:

| Header | Value | Purpose |
|--------|-------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Forces HTTPS (HSTS) |
| `X-Frame-Options` | `DENY` | Blocks iframe embedding (legacy browsers) |
| `X-Content-Type-Options` | `nosniff` | Prevents MIME-type sniffing |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limits referrer leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=(), payment=()` | Locks down browser APIs |
| `Content-Security-Policy` | See `_headers` for full allowlist | Controls allowed resource origins |
| `frame-ancestors 'none'` | Part of CSP | Modern clickjacking prevention |

### CSP rules
- No `'unsafe-inline'` in `style-src` — use CSS classes (`.is-visible` pattern) instead of inline styles in JS
- All new external image domains (e.g. TMDB posters, new CDNs) must be added to `img-src` in `_headers`
- `<script type="application/ld+json">` is data, not a script — CSP-exempt

### JS security practices (in `main.js`)
- All data inserted via `innerHTML` must pass through `escapeHtml()` — prevents XSS
- All URLs used in `href`/`src` attributes must pass through `safeUrl()` — prevents `javascript:` injection
- These two helpers must remain in place whenever new card types or data sources are added
- **Never use `onerror="..."` inline event attributes** — treated as `'unsafe-inline'` by CSP `script-src`; blocked by the site's own headers
- After every `innerHTML` assignment on a container with images, call `attachImageFallbacks(container)`
  - Images that should fall back to the podcast cover: add `data-fallback="${ARTWORK_FALLBACK}"` to the `<img>` tag
  - Images that should be removed on failure (e.g. teaser posters): add `data-fallback="remove"`
  - `ARTWORK_FALLBACK` constant at top of `main.js` holds the self-hosted podcast cover URL (`logo.jpg`)

---
> Source: [androidua/couchlobsters-website](https://github.com/androidua/couchlobsters-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
