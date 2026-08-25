## job-pulse

> Guidance for working in this repo. For deep detail see `SYSTEM_DOCUMENTATION.md`

# CLAUDE.md

Guidance for working in this repo. For deep detail see `SYSTEM_DOCUMENTATION.md`
(components, DB schema, troubleshooting), `SCHEDULING.md` (per-job scheduler),
`README.md` (overview), and `DashboardApp/DESIGN.md` (UI theme).

## What this is

Automated job aggregation for junior developers. Scrapes 400+ Israeli tech
companies across 8 ATS platforms, filters junior-suitable roles with an LLM
(Groq / GPT-OSS 20B, `openai/gpt-oss-20b`), emails a daily digest, and serves a Flask monitoring +
analytics dashboard. Data lives in Supabase (Postgres). Deployed on Render
(cron job + web service).

## Layout

```
Scrapers/            # scraping pipeline (the backend)
  CleanScript.py       # entry point / orchestrator, thread pool, --job flag
  job_scrapers.py      # 8 ATS API adapters
  telegramInsertBot.py # location filter, dedup, LLM enrich, email
  db_operations.py     # Supabase read/write + scheduler queries
  local_llm_function.py# Groq wrapper + shared classification prompt
  groq_batch_queue.py  # queues rate-limited jobs to Supabase Storage (JSONL)
  company_discovery.py # discovery CLI: discovery_search.py + discovery_ats.py
  alerting.py, log_cleanup.py, schedule_manager.py
  tests/               # pytest
DashboardApp/         # Flask dashboard + portfolio analytics
  app.py               # routes incl. GET /api/analytics/portfolio, /api/cron-trigger
  analytics.py, standardization.py  # read-time analytics (do NOT mutate DB)
  static/, templates/  # "Egg" light theme, token-driven CSS (see DESIGN.md)
  tests/
tools/scalefox_ingestion/  # one-off ingestion scripts, NOT runtime — nothing imports them
airflow_processes/data/combined_company_data3.json  # local company fallback
migrations/           # Supabase SQL
```

## Commands

```bash
# Scraper (run from Scrapers/)
RUN_MODE=local python CleanScript.py   # dashboard + infinite loop (dev)
RUN_MODE=cron  python CleanScript.py   # single run then exit (Render)
python CleanScript.py --job regular_ats # run one scheduled job manually

# Discovery (find new companies)
python company_discovery.py --dry-run

# Tests
pytest Scrapers/tests -v
python -m pytest DashboardApp/tests -v

# Groq batch queue smoke (needs service-role key)
python Scrapers/groq_batch_queue.py --smoke
```

## Scheduling model

One Render cron fires every 2h (`0 8-22/2 * * *`, skips Saturday). It does NOT
scrape everything each tick — the `scraper_schedule` table decides which of
three jobs is due: `regular_ats` (IL, ~110min), `new_ats` (iCIMS/Jobvite, IL,
660min), `usa_digest` (US only + email, 1380min). `next_run_at` advances the
moment a job starts (in `set_job_running`), preventing overlap. Full detail in
`SCHEDULING.md`.

## Conventions / gotchas

- **Two Supabase env-var spellings coexist:** `SUPABASE_URL`/`SUPABASE_KEY` and
  `supabaseUrl`/`supabaseKey` (scraper modules). Storage writes need
  `SUPABASE_SERVICE_ROLE_KEY` (server-only, never a public bucket).
- **Analytics is read-only.** `standardization.py` cleans messy rows at request
  time; it never backfills or mutates Supabase.
- **Frontend cache-bust:** when editing `styles.css` or `dashboard.js`, bump the
  `?v=` query on the `<link>`/`<script>` tags in `templates/index.html`.
- **ATS adapters return `(jobs_list, [])` tuples** — keep that shape. Lever
  auto-tries lowercase + EU API; don't special-case it.
- **LLM fallback never drops jobs** — if Groq is down/rate-limited, keyword
  matching classifies and everything still gets emailed; rate-limited jobs are
  queued to Storage for later batch processing.
- **One detail-page LLM call extracts desc, reqs, junior fit, city, and country.**
  Comeet/Greenhouse/Workday have specialized extractors; other ATS types use
  the generic JSON-LD/body extractor. Keep `location_raw` as the ATS source.
- Windows dev machine, PowerShell primary shell. Render runs Linux.

## Full env var reference

See `SYSTEM_DOCUMENTATION.md` (Configuration section) and `README.md`.

---
> Source: [NivDotan/Job-Pulse](https://github.com/NivDotan/Job-Pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
