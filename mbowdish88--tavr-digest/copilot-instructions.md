## tavr-digest

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md — tavr-digest

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**The Valve Wire** is a production daily automation pipeline that aggregates structural heart disease news — covering transcatheter AND surgical approaches to aortic (TAVR/TAVI), mitral (MitraClip, PASCAL, TMVR), and tricuspid (TriClip, TTVR) valves — from academic, regulatory, financial, and social sources. It uses Claude AI to synthesize a daily newsletter, publishes to a companion website, generates a weekly podcast, and is monitored via a Telegram bot.

**Goal:** Commercialization — paid subscriptions, sponsorships, and evolving into a platform that can spin up automated digests for any niche.

**Website:** The Valve Wire website lives inside this repo at `site/` (Next.js, deployed on Vercel from `site/` root directory). The original standalone repo was renamed to [medweb-template](https://github.com/mbowdish88/medweb-template) and serves as a reusable template.

**Live site:** [thevalvewire.com](https://thevalvewire.com)

## Running the Project

```bash
cd ~/projects/tavr-digest
source venv/bin/activate
python main.py              # Daily digest
python main.py --weekly     # Weekly summary
python main.py --podcast    # Podcast generation
```

Install dependencies: `pip install -r requirements.txt`

The pipeline runs daily at 6 AM Central via GitHub Actions. Manual trigger: `gh workflow run daily-digest.yml`

### Running the Website Locally

```bash
cd ~/projects/tavr-digest/site
npm install
npm run dev        # http://localhost:3000
npm run build      # production build
```

## Architecture

The pipeline follows: **Index Papers → Sources → Dedup → Summarize → Deliver → Write to site/**

### Sources (`sources/`)
Each module fetches from one external API/feed and returns a list of article dicts (`url`, `title`, `content`, `date`, `source`). Each has isolated error handling — one failing source doesn't block others.

| Module | Source | Notes |
|--------|--------|-------|
| `pubmed.py` | NCBI eUtils API (ESearch/EFetch) | 50 articles + 20 clinical trials |
| `preprints.py` | bioRxiv/medRxiv API | Filtered by relevance |
| `journals.py` | 12 RSS feeds (JACC, NEJM, Circulation, JAMA, Lancet, JTCVS, ATS, EJCTS, EHJ) | feedparser |
| `news.py` | Google News RSS | Site-specific searches (TCTMD, CV Business, CMS, SHN) |
| `regulatory.py` | FDA RSS feeds | Filtered by valve-related keywords |
| `trials.py` | ClinicalTrials.gov API v2 | 16 landmark trials monitored by NCT ID |
| `stocks.py` | yfinance | EW, MDT, ABT, BSX, AVR.AX — charts saved as PNGs |
| `social.py` | Nitter RSS bridges | 12 curated accounts — designed to fail gracefully |
| `financial.py` | SEC EDGAR 8-K filings + Google News financial RSS + yfinance news | |

### Processing (`processing/`)
| Module | Purpose |
|--------|---------|
| `dedup.py` | SQLite-backed deduplication (SHA256 URL hashing). Articles marked seen ONLY after successful email delivery. 90-day cleanup. Trials and stocks are live data, never deduped. |
| `summarizer.py` | Claude API (Sonnet 4, 16k token budget, 300s timeout) generates HTML newsletter. Includes major meeting detection for ACC, AHA, TCT, ESC, AATS, STS, EACTS. Also extracts structured article sidecar (key numbers, study design, journal) from raw inputs for downstream accuracy. |
| `weekly.py` | Compiles 7 days of daily digests into a single weekly summary. Saves both HTML and structured JSON sidecars. |

### Delivery (`delivery/`)
| Module | Purpose | Status |
|--------|---------|--------|
| `emailer.py` | SMTP email with HTML+plain-text via Jinja2 templates. Retries once with 10s delay. | Active |
| `website.py` | Writes structured JSON directly to `site/public/data/` (local filesystem, no API). Executive summary and key points preserve hyperlinks for verifiability. | Active |
| `site.py` | Publishes to GitHub Pages archive (`docs/`) | Active |
| `beehiiv.py` | Beehiiv API v2 newsletter delivery | Dormant |
| `substack.py` | Substack integration | Dormant |

### Website (`site/`)
The Valve Wire website — Next.js 16, App Router, Tailwind CSS 4, deployed on Vercel.

| Route | Purpose |
|-------|---------|
| `/` | Homepage — stock ticker, executive summary, key points, articles by section, podcast |
| `/archive` | Searchable/filterable article archive |
| `/weekly` | Latest weekly summary |
| `/podcast` | All podcast episodes with audio players + RSS |
| `/about` | Mission, editorial stance, AI disclosure |

Data flow: Pipeline writes JSON → `site/public/data/latest.json` + `site/public/data/digests/` → `git push` → Vercel auto-deploys.

All content pages use `dynamic = "force-dynamic"` for fresh data on each request.

### Podcast (`podcast/`)
Weekly podcast generation pipeline, triggered after the weekly digest completes.

| Module | Purpose |
|--------|---------|
| `scriptwriter.py` | Claude generates a two-host podcast script with journal hierarchy, meeting highlights, accuracy safeguards, and hallucination validation. Retries on JSON parse failure. 300s timeout. |
| `synthesizer.py` | OpenAI TTS voice synthesis (Nolan: "fable", Claire: "nova"). 3 retries with exponential backoff per segment. 120s timeout. |
| `assembler.py` | pydub MP3 assembly with intro/outro/background music |
| `show_notes.py` | HTML + Markdown show notes with timestamps |
| `transcriber.py` | Whisper API transcription. 300s timeout. |
| `publisher.py` | Publishes to GitHub Releases + RSS feed |
| `generate_assets.py` | Cover art and SVG generation |
| `audio_processing.py` | Audio utilities (HP/LP filter, compression, normalization) |

### Knowledge Base (`knowledge/`)
Injected into ALL Claude prompts for clinical context.

| File/Directory | Contents |
|----------------|----------|
| `papers/` | 52 indexed landmark study PDFs (PARTNER, COAPT, TRILUMINATE, etc.) |
| `papers/inbox/` | Drop new PDFs here — auto-indexed on next pipeline run |
| `papers_index.json` | Structured metadata for all papers (title, authors, journal, key finding, trial name) — loaded into every Claude prompt |
| `indexer.py` | Scans inbox, extracts text via PyMuPDF, uses Claude to extract metadata, renames files descriptively, updates papers_index.json |
| `guidelines/` | ACC/AHA 2020 + ESC 2025 valve guidelines (PDFs + extracted text) |
| `guidelines_index.json` | Structured recommendations by valve type with guideline disagreements |
| `guidelines_summary.md` | Full readable summary — loaded untruncated into every Claude prompt |
| `opinions/` | Expert consensus documents |

#### Paper Inbox Workflow
1. Download PDFs (e.g., from Cedars-Sinai with institutional access)
2. Drop them into `knowledge/papers/inbox/`
3. Next daily pipeline run auto-indexes them (or run `python -m knowledge.indexer`)
4. Indexer extracts text, uses Claude to get metadata, renames files descriptively
5. Papers are added to `papers_index.json` and injected into all future prompts

See `knowledge/papers/TAVR_Paper_Search_Prompt.md` for the comprehensive search prompt used to build the paper library (27 authors, 10 topic searches, 11 journals).

### Monitoring & Bots
| File | Purpose |
|------|---------|
| `monitor.py` | Analyzes workflow logs, sends alerts via email/Telegram |
| `telegram_bot.py` | Telegram command handler (/status, /logs, /cost, /rerun, /fix, /help) |
| `bot_server.py` | Always-on conversational Telegram bot powered by Claude (hosted on Railway) |
| `daily_summary.py` | Daily summary generator |

### Configuration (`config.py`)
Central config loading from `.env`. Defines 100+ search terms (aortic/mitral/tricuspid), 12 journal RSS feeds, 12 social accounts, 5 stock tickers, 16 landmark trial NCT IDs, SEC EDGAR CIKs, and all API settings.

## Daily Pipeline Flow (main.py)

1. **Validate API keys** (fail fast if missing)
2. **Index new papers** from `knowledge/papers/inbox/`
3. Fetch from all 10 sources (isolated error handling)
4. Deduplicate via SQLite
5. **Extract structured article sidecar** (key numbers, study design, journal from raw data)
6. Summarize with Claude API (with full guidelines + papers context)
7. **Save daily HTML + structured JSON sidecar** for weekly compilation
8. Publish to GitHub Pages (`docs/`)
9. **Write structured JSON to `site/public/data/`** (local filesystem)
10. Send email via SMTP
11. Mark articles as seen (ONLY after successful email send)
12. Cleanup dedup entries >90 days old
13. `git commit && git push` — Vercel auto-deploys website

## Weekly Pipeline Flow

1. Compile 7 days of daily digests (runs Saturday 6 AM CT)
2. Generate weekly HTML summary
3. Email weekly digest
4. Trigger podcast pipeline:
   - Load structured article sidecars as ground truth
   - Generate script (with meeting highlights, journal hierarchy, accuracy safeguards)
   - Synthesize audio (3 retries per segment, exponential backoff)
   - Assemble MP3
   - Generate show notes
   - Transcribe
   - Publish to GitHub Releases + RSS

## Anti-Hallucination Architecture

The pipeline has three layers of AI summarization, each introducing potential inaccuracy:
- **Layer 1:** Raw articles → Daily digest (~90% fidelity)
- **Layer 2:** Daily digests → Weekly digest (~80% fidelity)
- **Layer 3:** Weekly digest → Podcast script (~70% fidelity)

Safeguards in place:
- **Structured data sidecar** — key numbers, study design, journal extracted from raw articles via regex (no AI) and persisted alongside daily HTML
- **Raw metadata passthrough** — podcast scriptwriter receives verified article data as ground truth alongside the summarized weekly HTML
- **Hallucination validation** — `_validate_references()` checks study names, percentages, journal attributions, and p-values against source material
- **Accuracy safeguards in prompts** — explicit instructions to never fabricate numbers, journals, or authors
- **Verifiable hyperlinks** — executive summary and key points preserve source links
- **Full guidelines context** — 16,922 chars of guidelines + 9 guideline disagreements loaded into every prompt (never truncated)
- **52 indexed landmark papers** — loaded into every prompt via papers_index.json

## Journal Hierarchy

All Claude prompts prioritize findings from higher-impact journals:
**NEJM > JAMA > JACC > Lancet > EHJ > JACC:CI > ATS, JTCVS, EJCTS**

NEJM or JAMA publications should ALWAYS be highlighted in executive summaries and podcast top stories.

## GitHub Actions (`.github/workflows/`)

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `daily-digest.yml` | Daily 6 AM CT + manual | Main pipeline (commits `site/public/data/` for Vercel) |
| `weekly-digest.yml` | Saturday 6 AM CT + manual | Weekly summary |
| `weekly-podcast.yml` | After weekly-digest | Podcast generation + publishing |
| `daily-summary.yml` | Scheduled | Summary generation |
| `monitor.yml` | Scheduled | Pipeline monitoring + alerts |
| `telegram-commands.yml` | Webhook | Telegram bot command handler |

All workflows restore/cache `seen_articles.db` for dedup continuity.

## Newsletter Sections (generated by Claude)
- Executive Summary (plain language, accessible to patients, with source hyperlinks)
- Aortic Valve (TAVR/TAVI)
- Mitral Valve (MitraClip, PASCAL, TMVR)
- Tricuspid Valve (TriClip, TTVR)
- Surgical vs. Transcatheter Comparisons
- Financial Analysis (SEC filings, M&A, reimbursement trends)
- Valve Industry Stocks (with market cap, P/E, target price, consensus)
- Clinical Trial Updates

## Key Design Decisions

- **Monorepo** — website lives in `site/`, pipeline writes JSON locally, single push deploys everything
- Each source module has isolated error handling; one failing source doesn't block others
- Claude API calls have 300s timeouts and retry logic
- TTS has 3 retries with exponential backoff (2s, 4s, 8s) + 120s timeout
- API keys are validated at startup (fail fast)
- Dedup marks articles as seen ONLY after successful email send
- `main.py` skips digest generation entirely if no new articles found after dedup
- **Daily digests are the FOUNDATION.** They must NEVER be deleted. Weekly digest, podcast, website archive, and all future features build on top of them.

## CRITICAL EDITORIAL STANCE

Many structural heart technologies have gotten ahead of the science and clinical guidelines. All content must be **BALANCED and CIRCUMSPECT**:
- Always present BOTH sides: favorable findings AND limitations/criticisms
- Flag study design weaknesses: non-randomized, industry-sponsored, short follow-up, small samples
- Never present transcatheter superiority as settled when long-term data is lacking
- Reference critical perspectives (Bowdish, Badhwar, Mehaffey, Kaul, Miller, Chikwe)
- Tone: expert skepticism — enthusiastic about real advances but always questioning

## Infrastructure

| Component | Platform |
|-----------|----------|
| Monorepo | github.com/mbowdish88/tavr-digest |
| Template repo | github.com/mbowdish88/medweb-template |
| Domain | thevalvewire.com (Cloudflare DNS → Vercel) |
| Website hosting | Vercel (Hobby plan, deploys from `site/` root) |
| Telegram bot | Railway (always-on) |
| CI/CD | GitHub Actions |
| Local path | `~/projects/tavr-digest` |

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents

### 3. Self-Improvement Loop
- After ANY correction from the user: update tasks/lessons.md with the pattern

### 4. Verification Before Done
- Never mark a task complete without proving it works

### 5. Autonomous Bug Fixing
- When given a bug report: just fix it

## Core Principles

- Simplicity First: Make every change as simple as possible. Minimal code impact.
- No Laziness: Find root causes. No temporary fixes. Senior developer standards.
- Minimal Impact: Only touch what's necessary. No side effects with new bugs.
- Accuracy First: Never fabricate citations, numbers, or study names. Everything must be verifiable.

## gstack

[gstack](https://github.com/garrytan/gstack) is installed globally at `~/.claude/skills/gstack/` — no per-project setup needed. Its skills are available in any Claude Code session on this machine.

**Use the `/browse` skill from gstack for ALL web browsing.** Never use `mcp__claude-in-chrome__*` tools.

**Available gstack skills:**
/office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review, /design-consultation, /design-shotgun, /design-html, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse, /connect-chrome, /qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /retro, /investigate, /document-release, /codex, /cso, /autoplan, /plan-devex-review, /devex-review, /careful, /freeze, /guard, /unfreeze, /gstack-upgrade, /learn

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mbowdish88) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
