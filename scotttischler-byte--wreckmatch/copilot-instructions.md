## wreckmatch

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## GEO / AI visibility (automated — all properties)

**Secrets:** `.secrets-setup` at repo root (`CRON_SECRET`, `INDEXNOW_KEY`) — same file as `../injuredhelp.ai`. Scripts auto-load; **never ask the user to re-paste.**

| Property | Domain |
|----------|--------|
| WreckMatch | www.wreckmatch.com |
| Accident Survival Guide | www.accidentsurvivalguide.com |
| Bobby Garcia | www.bobbygarcia.com |
| InjuredHelp | injuredhelp.ai |

| Command | Purpose |
|---------|---------|
| `npm run geo:run` | Full GEO automation + prod cron ping |
| `npm run seo:indexnow` | Sitemap IndexNow all hosts |
| `python3 scripts/geo_automation.py` | Audits + report |

Docs: `docs/CURSOR_SECRET_SAUCE.md`, `docs/GEO_AUTOMATION.md`, playbook `/secret-sauce.html`

Production blog autopilot (50 states/day): **`../injuredhelp.ai`** — see `../injuredhelp.ai/docs/AGENT_BLOG_AUTOPILOT.md`

## SEO 24/7 agents

Cloud automation (no laptop required) runs via GitHub Actions:

| Workflow | Schedule | Command equivalent |
|----------|----------|-------------------|
| `geo-automation.yml` | 2×/day | `python3 scripts/geo_automation.py` |
| `seo-agent-hourly.yml` | Every hour | `npm run seo:agent` |
| `seo-agent-6h.yml` | Every 6 hours | `node scripts/seo-agent.mjs full` + autopilot |
| `publish-seo.yml` | Daily 14:00 UTC | `npm run seo:daily` + blog publish |
| `autopilot-blog-bulk.yml` | 3× daily | 279-city queue (needs `OPENAI_API_KEY`) |

Local loop (machine must stay on): `npm run seo:agent:loop` or Cursor `/loop 30m bash scripts/run-seo-agent-loop.sh`

Status: `npm run seo:agent:status` — log at `content/agents/seo-agent-log.jsonl`

Do not bulk-publish thin content.

---
> Source: [scotttischler-byte/wreckmatch](https://github.com/scotttischler-byte/wreckmatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
