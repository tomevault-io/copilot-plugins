## seo-plugin

> This repo is a Claude Code plugin. It is **SEO analysis for e-commerce stores and the

# seo-engine — guidance for Claude Code

This repo is a Claude Code plugin. It is **SEO analysis for e-commerce stores and the
agencies that run them**. One `/seo` command, eight focused sub-commands, a hybrid
Python engine, parallel sub-agents, a 0-100 score, and a white-label PDF.

## Architecture

- `skills/seo/SKILL.md` — the orchestrator. Routes commands; for `audit`/`report` it
  fans out parallel sub-agents and synthesizes.
- `skills/seo-*/SKILL.md` — one per command: page, product, schema, geo, content,
  technical, report.
- `agents/*.md` — the sub-agents the orchestrator spawns in parallel (technical,
  product, schema, content, geo).
- `scripts/*.py` — the engine. Skills/agents call these; they do the deterministic work.
  - `firecrawl_fetch.py` — **the fetcher (Firecrawl, required).** Emits a capture JSON
    `{url, rawHtml, markdown, metadata}` to a file (prints only the path — the huge HTML
    never enters the conversation). Reads `FIRECRAWL_API_KEY` via `seo_config.py`.
  - `seo_config.py` — loads API keys from `~/.seo-engine/.env` (GUI apps don't inherit
    your shell env), with real env vars as fallback.
  - `parse_html.py` — on-page SEO elements → JSON. Primary mode `--capture <file>`:
    head signals from metadata + raw `<head>`, structure from `<body>`, JSON-LD from
    `rawHtml`, content word count from main-content markdown.
  - `schema_validate.py` — Schema.org coverage, deep Product/Offer checks (`--capture`).
  - `pagespeed_check.py` — Core Web Vitals via PageSpeed Insights, takes a `<url>`
    (Google fetches the page, so no Firecrawl needed here). Key optional.
  - `pdf_report.py` — Markdown → white-label PDF.
  - `url_safety.py` — SSRF guard for the direct URL calls (PageSpeed).

## Fetching (Firecrawl is required)

Real stores block plain HTTP (Cloudflare/bot protection). **Never reintroduce a
`requests`/Playwright page fetcher** — it gets 403'd and produces false data. Always
fetch via `firecrawl_fetch.py` (it returns a small JSON path, keeping multi-MB pages out
of the conversation — critical for reliability). Parse `rawHtml`, not Firecrawl's cleaned
`html` (which strips `<head>` meta, canonical, and JSON-LD → false negatives). PageSpeed
takes a URL directly; robots.txt/sitemap fetch with curl.

## Voice (non-negotiable in all user-facing output)

Operator-first. Direct. Short sentences. Consultant-to-founder, never guru. Lead with
what's broken and what it costs. Every recommendation is concrete and falsifiable. No
hype, no filler, never pad a report to look bigger. Light profanity is fine when
something's genuinely broken.

## Hard rules

- **Real data only.** Every finding traces to a script result or something read on the
  page. No invented metrics, keyword volumes, or backlink/rank data (those need paid
  APIs this plugin doesn't ship — flag them out of scope).
- **Never fabricate review/rating schema.** It's a manual-action risk, not a hack.
- **Respect the ~100-page audit gate.** On large sites, sample templates instead of
  crawling everything.
- **White-label means white-label.** If a brand is supplied for a report, `seo-engine`
  appears nowhere in the client-facing output.

## Dev notes

- Python 3.9+. `./install.sh` builds `.venv` and installs deps + the Playwright browser.
- Scripts are individually runnable for testing: `python scripts/parse_html.py --url <url>`.
- Attribution: the orchestrator + parallel-agent pattern was inspired by AgriciDaniel's
  MIT-licensed `claude-seo`. Independent implementation; see LICENSE.

---
> Source: [mikefutia/seo-plugin](https://github.com/mikefutia/seo-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
