## readit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Readit is a personal read-it-later system built entirely on GitHub infrastructure (Actions, Pages, Repo as DB). Users save articles via iOS Shortcuts, GitHub Actions auto-fetches content, generates summaries/translations via Claude API, and produces a daily curated podcast via OpenAI TTS. The frontend is static HTML served by GitHub Pages.

## Architecture

- **No server, no database** — GitHub Repo stores data (JSON + Markdown), Actions handle compute, Pages serves the frontend, Secrets manages API keys
- **Runtime**: Node.js (runs inside GitHub Actions)
- **Frontend**: Static HTML, no framework, system fonts, auto Light/Dark Mode (reference: Reeder/Instapaper style)

### Three GitHub Actions Workflows

1. **save-article** — Triggered by `repository_dispatch` (from iOS Shortcut) or `workflow_dispatch`. Writes entry to `data/articles.json` with status `pending`.
2. **process-articles** — Triggered by save-article completion (chain) + cron every 30min. Fetches content via Jina Reader API, generates summary/translation via Claude API, writes Markdown files, runs `build-site.js`.
3. **daily-curation** — Cron at UTC 23:00. Summarizes last 24h articles via Claude, generates audio via OpenAI TTS, uploads MP3 to Alibaba Cloud OSS, updates `data/episodes.json`, runs `build-site.js`.

### Data Flow

```
iOS Shortcut → GitHub API (repository_dispatch)
  → save-article.js → articles.json (status: pending)
  → process-articles.js → Jina fetch (→ fetched) → Claude summary/translate (→ ready)
  → build-site.js → static HTML in site/
  → GitHub Pages auto-deploy
```

### Article Status Machine

`pending` → `fetched` → `ready` (happy path)
`pending` → `error` (Jina fails; no auto-retry)
`fetched` stays `fetched` if Claude fails (auto-retries on next run)

## Planned File Structure

```
scripts/           — Node.js scripts run by Actions
  save-article.js, process-articles.js, daily-curation.js, build-site.js
data/              — JSON indexes + Markdown article content
  articles.json, episodes.json, articles/{id}.md, articles/{id}.zh.md
site/              — Static HTML output (GitHub Pages root)
  index.html, article.html, curation.html, style.css
.github/workflows/ — Action workflow YAML files
```

## Key Design Decisions

- **Jina Reader** is the sole content fetcher (handles JS-rendered pages, Twitter/X)
- **Immersive translation** for English articles: blockquote = original English, normal text = Chinese translation (Markdown format, rendered directly by frontend)
- **Idempotent processing**: Actions check article status, only process incomplete items, safe to re-run
- **Audio stored on Alibaba Cloud OSS**, not in the repo (keeps repo small)
- Article IDs follow format: `YYYYMMDD-{random}` (e.g., `20260309-a1b2c3`)

## External APIs & Secrets

| Secret | Service |
|--------|---------|
| OPENROUTER_API_KEY | OpenRouter API (routes to Claude Sonnet) — summaries, translation, curation |
| OPENAI_API_KEY | OpenAI TTS API — podcast audio |
| OSS_ACCESS_KEY_ID, OSS_ACCESS_KEY_SECRET, OSS_REGION, OSS_BUCKET | Alibaba Cloud OSS — audio storage |

iOS Shortcut uses a separate GitHub PAT (fine-grained, `contents: write` only).

## Development Commands

```bash
npm install              # install dependencies
node scripts/save-article.js       # save a new article (needs URL input)
node scripts/process-articles.js   # process pending/fetched articles
node scripts/daily-curation.js     # generate daily podcast
node scripts/build-site.js         # rebuild static site into site/
```

## Development Phases

- **Phase 1** (core): save → fetch → read pipeline + basic Pages site + iOS Shortcut
- **Phase 2**: daily curation podcast (Claude summary + TTS + OSS)
- **Phase 3**: UX polish, bilingual toggle, PWA, client-side search

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tinyao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
