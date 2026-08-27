## travel-blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static HTML travel blog for exitfloridakeys.com, hosted on GitHub Pages. Uses vanilla HTML/CSS/JavaScript with Node.js automation scripts.

**Content Focus:** Travel-only (no finance, health, entertainment, or hobbies topics).

**Content Pipeline:** n8n workflows (local, gitignored) → Airtable → GitHub → GitHub Pages

## Common Commands

All automation runs via GitHub Actions when pushing to `articles/**`. To run scripts locally:

```bash
# Update search index
node scripts/update-search-index.mjs

# Update homepage and latest articles pages
node scripts/update-latest-posts.mjs

# Inject sidebars into article pages
node scripts/update-sidebar.mjs

# Update author profile pages
node scripts/update-author-pages.mjs

# Inject secondary placeholder images
node scripts/inject-secondary-figures.mjs

# Update site navigation
node scripts/update-site-nav.mjs

# Fix Schema.org date format in articles (idempotent, skips already-fixed files)
node scripts/fix-schema-dates.mjs            # dry-run
node scripts/fix-schema-dates.mjs --apply    # apply

# Replace fallback hero images with relevant Unsplash photos
node scripts/fix-fallback-images.mjs            # dry-run
node scripts/fix-fallback-images.mjs --apply    # apply

# Make Google Fonts non-render-blocking in articles
node scripts/fix-font-loading.mjs               # dry-run
node scripts/fix-font-loading.mjs --apply       # apply
```

No local package.json is tracked. CI workflows install dependencies inline (PostCSS, Terser, Fuse.js).

## Architecture

### Content Structure
- `articles/` - HTML article files (kebab-case naming)
- `authors/` - Author profile pages
- `assets/styles/` - CSS (minified in CI)
- `assets/scripts/` - Client-side JS modules (minified in CI)
- `assets/team/` - Author avatar images (96x96 WebP, displayed at 24x24 CSS pixels with 4x retina). Must have `width="24" height="24"` in HTML.
- `assets/search.json` - Auto-generated search index consumed by Fuse.js
- `workflows/` - n8n workflow JSON files (gitignored, local only)

### n8n Workflows (Local)
Two main workflows handle content generation:

1. **Topic Generator** (`loVBaBy9K6JYCQMI`) - Fetches Google Trends, generates travel topics, creates Airtable records
2. **Earnest HemmingwAI** (`26JTEvvmpO2nAf48`) - Full article pipeline:
   - Research phase: Scrapes Reddit, Google Maps reviews, Wikivoyage for real traveler data
   - AI generates chapters using research insights (specific places, prices, honest warnings)
   - Builds HTML article from template
   - Sets status to "Review" for human approval before publishing

**Required APIs:** SerpAPI (Google Maps reviews), OpenAI, Airtable. Reddit and Wikivoyage are free.

**Airtable:** Base `app94N1Rtl3jobWLe`, Table `tblfhOlFljHhmtzNu` (Arbitrage). Fields: Topic, Category, Status, Author, Keywords, Destination, Url, Sentiment.

### Automation Scripts (`/scripts`)
Build scripts use ES Modules (`.mjs`) with Node.js native APIs. They perform regex-based HTML manipulation to inject content:
- Sidebars with recent/hot articles
- Secondary placeholder images from Picsum.photos
- Navigation headers across all pages
- Author article lists

Shared utilities live in `/scripts/lib/`.

### GitHub Actions Workflows
Push to `articles/**` triggers:
1. `update-search-index.yml` - Runs sidebar, figures, search, author, latest posts, and nav scripts
2. `update-latest-posts.yml` - Updates index.html and article listing pages
3. `generate-sitemap.yml` - Regenerates sitemap.xml/txt with article timestamps
4. `minify.yml` - Compresses CSS (PostCSS) and JS (Terser)

### Article Metadata
Scripts extract metadata from HTML using regex:
- Title: `<meta property="og:title">` (strips "Travel Guide |" prefix)
- Description: `<p itemprop="description">`
- Author: `<meta name="author">`
- Date: `<time datetime="...">`
- Image: First `<figure><img>` src/alt

### Client-Side Features
- Search powered by Fuse.js reading from `assets/search.json`
- Lazy loading on all images
- Responsive images with srcset (WebP, AVIF, JPG formats)

## Conventions

- Article filenames: kebab-case slugs (e.g., `african-safari-experience.html`)
- Author filenames: lowercase with hyphens (e.g., `frank-axton.html`)
- Feature branches: `codex/*` for manual changes
- Automated branches: `automated/sitemap-update`
- All scripts use ES Modules syntax
- HTML manipulation uses regex patterns (no DOM parsers)
- Bulk HTML edits across all articles: use `perl -i -0pe` (slurp mode) via Bash, not the Edit tool
- Article workflow status: Todo → In Progress → Review → Done (human review required)

## Deployment

Master branch deploys to GitHub Pages. Custom domain configured via CNAME file.

## Troubleshooting

### n8n Workflow Issues
- **Duplicate articles:** Check Get Articles node filter - must include all statuses and fetch Destination field
- **Duplicate angle detection:** Validate Angle node (Code) compares extracted angle categories against existing articles. Angles: food, hidden_gems, budget, crowds, culture, adventure, luxury, practical, history, nature. Overrides AI's `isDifferentFromExisting` if angle already covered.
- **Empty images:** Earnest HemmingwAI Build Article node needs FALLBACK_IMAGE for when Unsplash returns no results
- **n8n Code nodes:** `fetch()` is not available in the sandbox. Use `$http.request()` instead.
- **Unsplash image dimensions:** `urls.regular` serves `?w=1080`. HTML `width`/`height` must use display dimensions (e.g., 1080x720), NOT native photo dimensions (e.g., 6000x4000).
- Use `mcp__n8n__n8n_get_workflow` with `mode: "full"` to inspect node parameters
- n8n MCP workflow output is very large (~150KB). Use `mode: "structure"` for overview, or parse the saved JSON file with `python3`/`jq`. Response structure: `[{type: "text", text: "{success, data: {nodes: [...]}}"}]`

### SEO Requirements
Articles must include:
- Schema.org Article JSON-LD with `publisher`, `dateModified`, and `author.url`. Dates must use full ISO 8601 with timezone (e.g., `2025-10-11T20:01:48.364Z`), not date-only (`2025-10-11`)
- BreadcrumbList JSON-LD (Home → Articles → Article Title)
- Open Graph `article:published_time`, `article:modified_time`, `article:author`
- Twitter handle: `@Memopolis228816`

### Performance Requirements

- Author avatars: 96x96 WebP (~2-5 KB), `width="24" height="24" loading="lazy"` in HTML
- Logo: `width="114" height="64"` in HTML
- Sidebar random image: `width="200" height="200" loading="lazy"` in HTML
- Hero images: `width="1080" height="720"` in HTML, `fetchpriority="high"`
- Google Fonts: non-blocking (`preload` + `media="print"` + `onload="this.media='all'"` pattern)
- Footer links: `#4da3ff` on black for 4.5:1+ contrast; in-article links must have `text-decoration: underline`

---
> Source: [Gh05d/travel-blog](https://github.com/Gh05d/travel-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
