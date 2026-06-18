## agent-seo-blueprint

> Comprehensive SEO engine — run keyword/niche research, programmatic-SEO and content planning, technical+on-page+content site audits, link-building/authority campaigns, and rank/traffic monitoring, all grounded in a distilled SEO methodology and driven by live data (Ahrefs, Google Search Console, GA4, live SERP, PageSpeed/Lighthouse). Use whenever the user wants to do SEO work or asks what the course says about a topic — finding/validating a niche, keyword research, building a content/sitemap plan, auditing a site's SEO, getting backlinks, programmatic SEO, free-tool pages, on-page optimization, surviving Google updates, measuring if SEO is paying off, or summoning customer personas to ideate content angles. Works on any site; pulls real data via API or browser fallback.


# Agent SEO Blueprint

A complete, agentic SEO skill: a **distilled methodology** (mirrored from a full SEO course, searchable) plus
**runnable pipelines** that execute it against live data and write artifacts into a per-project workspace.

Two things any agent can do here:
1. **Answer SEO questions grounded in the methodology** — "what does the course say about keyword finding / programmatic SEO / backlinks?"
2. **Do the work end-to-end** — research, content planning, audits, link building, monitoring — using Ahrefs, GSC, GA4, live SERP, and PageSpeed.

## How to use this skill (progressive disclosure)

Load only what the task needs. The flow is almost always:

1. **Resolve the workspace** (for any project work):
   `python3 scripts/workspace.py status --path <DIR>` → if none exists, **ask the user** before creating, then
   `python3 scripts/workspace.py init --path <DIR> --name <NAME> [--domain d] [--niche n]`.
2. **Pick the workflow** for the job (see table) and open that runbook.
3. The runbook tells you which **playbooks** to load for the method and which **scripts/integrations** to run.
4. Write outputs to the workspace via `scripts/report.py`.

Do NOT read every playbook up front. Use the workflow runbook + course search to pull only the relevant ones.

## Route by intent

| The user wants to… | Open this workflow |
|---|---|
| Find/validate a niche, do keyword research, plan a sitemap, find competitor gaps | `workflows/research-and-ideation.md` |
| Plan/produce content: programmatic SEO, free-tool pages, landing pages, articles, on-page | `workflows/content-production.md` |
| Audit a live site (technical + on-page + content) with a prioritized fix list | `workflows/site-audit.md` |
| Get backlinks / build authority (link stealing, HARO, affiliate, outreach, content rings) | `workflows/authority-and-links.md` |
| Monitor rankings/traffic, respond to Google updates, measure ROI | `workflows/monitoring.md` |
| Ask "what does the course say about X" / look up a method | Course search (below) |
| Ideate content angles / pressure-test a niche with customer personas | ICP personas (below) |

## Course knowledge: search + mirror

The full course is mirrored and searchable.

- **Search** (the "what does the course say about X" entry point):
  `python3 scripts/search_course.py "<query>"` → returns the matching lessons (with summaries) **and** the distilled
  playbooks to read. Add `--full` to also search transcript bodies, `--json` for machine output.
- **Mirror / outline:** `references/course-index/course-index.md` (all 62 lessons, 6 chapters, with links).
- **Index data:** `references/course-index/course-index.json` (per-lesson summary, key takeaways, playbook links).
- **Coverage proof:** `references/coverage-map.md` (every lesson → the playbook(s) that cover it).

## Knowledge layer — playbooks (`references/playbooks/`)

Distilled, original methodology grouped by area. Load the specific file a step needs.

- **research/** — keyword-fundamentals, search-intent, match-and-exceed, seo-metrics, keyword-research-tools,
  finding-and-validating-niches, competitor-research, keyword-to-sitemap, research-for-existing-sites
- **content/** — content-fundamentals, content-types-overview, programmatic-seo, free-tools-strategy, content-pages,
  landing-pages, articles, content-rings, on-page-optimization, content-what-not-to-do
- **authority/** — understanding-authority, link-stealing, affiliate-programs, acquiring-domain-authority, haro,
  manual-outreach, building-an-audience, content-rings-for-links, other-link-building, linkbuilding-what-not-to-do
- **foundations/** — seo-philosophy, seo-and-ai-future, seo-process-overview
- **maintenance/** — navigating-google-updates, keyword-intent-evolution, staying-ahead-with-backlinks, measuring-seo-results

## Data integrations (`references/integrations/` + `scripts/`)

Each source has a reference doc (API path + browser-fallback procedure) and a helper script. **Browser fallback is
performed by you via Chrome MCP tools** when no API key/creds are present — the doc gives the exact navigate/read steps.

| Source | Script | API? | Reference |
|---|---|---|---|
| Ahrefs | `scripts/ahrefs_client.py` | API if `AHREFS_API_KEY`, else browser | `references/integrations/ahrefs.md` |
| Google Search Console | `scripts/gsc_pull.py` | API if creds, else browser | `references/integrations/gsc.md` |
| GA4 | `scripts/ga4_pull.py` | API if creds, else browser | `references/integrations/ga4.md` |
| Live SERP | `scripts/serp_capture.py` | browser only (Chrome MCP) | `references/integrations/serp.md` |
| PageSpeed/Lighthouse | `scripts/pagespeed_run.py` | public API (key optional) | `references/integrations/pagespeed.md` |

Auth rule: the skill **never** enters passwords or completes OAuth itself — it directs the user to log in / authorize,
then reads the logged-in session. API keys come from env vars named in the workspace `project.json` (never stored in plaintext).

## ICP persona subagents (`agents/`)

Summon Opus subagents that role-play ideal customers to ideate and pressure-test ideas:
- `agents/icp-persona.md` — launcher template for ONE persona (fill the `{{PLACEHOLDERS}}`). Dispatch **3–6 in parallel**
  via the Agent tool with `model=opus` for divergent ideation.
- `agents/synthesizer.md` — merges the persona outputs into ranked content angles, intent-clustered keyword seeds, and a
  niche-validation verdict. Feeds `workflows/research-and-ideation.md`.
- `agents/personas/` — reusable persona library + examples.

## Per-project workspace

One folder per site: `project.json` (domains, niche, competitors, data-source config, monitoring settings) plus dated
artifacts in `audits/ keywords/ briefs/ monitoring/ outreach/ research/`. Managed by `scripts/workspace.py`; artifacts
written by `scripts/report.py` ⇄ templates in `assets/`.

## Guardrails

- **Irreversible actions** (publishing content, sending outreach/email, submitting forms, disavow uploads): DRAFT only,
  then get explicit user confirmation before acting. Never auto-send.
- **SERP/Google reads** are for personal research — modest volume, respect bot detection, never bypass CAPTCHAs.
- **Source material** in `_source/` is private paid-course content (gitignored). The playbooks are original distillations
  and are the only course-derived material safe to share.

## Build / maintenance notes

See `DESIGN.md` for the architecture. To extend: add a playbook under `references/playbooks/<area>/`, update its lesson
mapping in `references/course-index/course-index.json`, and reference it from the relevant workflow.

---
> Source: [StoicEnso/agent-seo-blueprint](https://github.com/StoicEnso/agent-seo-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
