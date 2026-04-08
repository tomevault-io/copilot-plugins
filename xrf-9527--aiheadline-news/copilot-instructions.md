## aiheadline-news

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AIHeadline.news is an automated AI news aggregation site built with Hugo and the Hextra theme. Content is dynamically generated from the `ai-briefing-archive` repository and deployed to both Cloudflare Workers and GitHub Pages.

**Key Architecture Principle**: `content/` and `source-news/` directories are **NOT** committed to git. They are dynamically generated during CI/CD builds by fetching and processing external news data.

## Development Commands

### Local Development
```bash
# Quick start: Sync content and start dev server
bash .github/scripts/dev.sh

# Manually fetch latest news data
./.github/scripts/update-source-news.sh

# Sync content to Hugo format
./.github/scripts/sync-news.sh --mode=auto

# Start Hugo development server
hugo server

# Build static site
hugo --gc --minify

# Start Cloudflare Worker development (requires .dev.vars with GA4_SERVICE_KEY)
npm run dev
```

### Content Sync Modes
- `--mode=auto` - Auto-detect: incremental if changes detected, else full
- `--mode=full` - Full rebuild of all content
- `--mode=incremental` - Only sync changed dates (use with --dates=YYYYMMDD,...)

## Architecture Deep Dive

### Content Generation Pipeline

1. **Source Fetch** (`update-source-news.sh`):
   - Clones/updates `ai-briefing-archive` repo into `source-news/`
   - Uses shallow clone (`--depth=1`) for efficiency
   - Stores metadata in `.source-news-meta`

2. **Content Transformation** (`sync-news.sh`):
   - Scans `source-news/YYYY/MM/source-slug/briefing_YYYYMMDDTHHMMSS*.md` files
   - Groups by date, selecting latest briefing per source per day
   - Generates Hugo frontmatter with:
     - Title: Localized date in Chinese (e.g., "10月17日 AI 快讯")
     - Weight: Calculated for reverse chronological sorting
     - Sources array: Display names extracted from H1 headers
   - Transforms markdown: Downgrades headers (H1→H2), localizes timestamps to Beijing time
   - Outputs to `content/YYYY-MM/YYYY-MM-DD.md`
   - Generates month index pages (`_index.md`)
   - Updates homepage with latest briefing

3. **Dual Build System**:
   - **Cloudflare Worker**: Built to `public-worker/` with base URL `https://aiheadline.news`
   - **GitHub Pages**: Built to `public-pages/` with dynamic base URL from Actions

### Hugo + Hextra Theme Integration

**Critical Theme Constraint**: All content pages inherit `cascade: type: docs` from `content/_index.md`. This affects:
- Hugo's `.RegularPages` collection (excludes docs-type pages in some contexts)
- RSS template lookup order
- Theme's sidebar and navigation generation

**RSS Feed Specifics**:
- **Template Location**: Must use `layouts/index.rss.xml` (NOT `layouts/_default/rss.xml`)
- **Page Collection**: Use `site.RegularPages` directly for homepage RSS
- **Content Output**: Configured with `rssFullContent = true` to include full HTML content
- **Item Limit**: 20 items (configured in `hugo.toml`)

### Cloudflare Worker Features

The `_worker.ts` file implements:
- Static asset serving via `env.ASSETS`
- GA4 Data API integration using JWT self-signed authentication
- Edge caching for analytics data (total visits + active users)
- Security headers and CORS handling

**Environment Variables Required**:
- `GA4_SERVICE_KEY` - Full GA4 service account JSON (single-line string)
- `GA4_PROPERTY_ID` - Numeric GA4 property ID
- `GA_START_DATE` - Optional start date for cumulative stats (YYYY-MM-DD)

## Common Issues & Solutions

### Empty RSS Feed
**Symptom**: `index.xml` contains channel info but no `<item>` elements
**Cause**: Wrong RSS template location or page collection method
**Solution**:
- RSS template MUST be at `layouts/index.rss.xml` (not `_default/rss.xml`)
- Use `site.RegularPages.ByDate.Reverse` for homepage feed
- Remember: content pages inherit `type: docs` from cascade

### Content Not Updating
**Symptom**: New briefings don't appear after deployment
**Cause**: Content sync script failed or source repo not updated
**Debug**:
1. Check GitHub Actions logs for `update-source-news.sh` step
2. Verify `sync-news.sh` output shows expected date processing
3. Confirm Hugo build shows correct page count (check "Pages │ XX" in build output)

### Hugo Module Errors
**Symptom**: Build fails with "module not found" for Hextra theme
**Solution**: Hugo automatically downloads modules on first build. Ensure:
- Hugo version ≥ 0.150.0 (extended)
- `go.mod` and `go.sum` are committed
- CI has network access to download `github.com/imfing/hextra`

## File Structure Notes

```
layouts/
├── index.rss.xml          # Homepage RSS feed (CRITICAL: must be this exact path)
├── _default/
│   ├── list.markdown      # Markdown output format
│   └── single.markdown
├── partials/
│   └── custom-head.html   # GA4 + AdSense injection
└── shortcodes/            # Custom Hugo shortcodes
```

**Git-Ignored Directories**:
- `content/` - Generated by sync-news.sh
- `source-news/` - Cloned from ai-briefing-archive
- `public/`, `public-worker/`, `public-pages/` - Build outputs
- `.cache/hugo_cache/` - Hugo build cache

## Deployment Flow

GitHub Actions triggers on:
- Push to `main` branch
- Daily cron at 00:00 UTC
- Manual workflow_dispatch

Build produces two independent artifacts:
1. **site-worker** → Deployed to Cloudflare Workers
2. **pages artifact** → Deployed to GitHub Pages

Both use the same Hugo source but different base URLs.

## Hugo Configuration Essentials

From `hugo.toml`:
- **Theme**: Hextra via Hugo modules (`github.com/imfing/hextra`)
- **Output Formats**: HTML, RSS, and custom Markdown format
- **RSS Limit**: 20 items (`services.rss.limit`)
- **RSS Full Content**: Enabled (`params.rssFullContent = true`)
- **Language**: Chinese (`zh-cn`) with CJK language support
- **Cascade Type**: All pages under `content/` inherit `type: docs`

## Experience from This Session

**RSS Feed Fix Journey**:
1. Initially tried `layouts/_default/rss.xml` - didn't work
2. Attempted filtering by `type: docs` - still empty
3. Simplified to `site.RegularPages` - no effect
4. **Solution**: Created `layouts/index.rss.xml` - immediately worked

**Lesson**: Hugo's template lookup order prioritizes page-kind-specific templates (`index.rss.xml`) over default templates. For homepage RSS with themes that use custom page types, use the explicit `index.rss.xml` path.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xrf-9527)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/xrf-9527)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
