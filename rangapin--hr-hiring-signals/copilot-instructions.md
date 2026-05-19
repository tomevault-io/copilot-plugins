## hr-hiring-signals

> A system that monitors Polish job boards for HR hiring activity and alerts when companies post 3+ HR roles (buying signal for Lyra Polska's wellbeing services).

# HR Job Market Alerter

## Project Overview
A system that monitors Polish job boards for HR hiring activity and alerts when companies post 3+ HR roles (buying signal for Lyra Polska's wellbeing services).

## Demo Deadline
Thursday Feb 13 — demo to Jagoda.

## Architecture
- **PyPI package** called `hr-alerter` with Click CLI
- **pypyr pipelines** for daily/weekly orchestration
- **5-dimension scoring engine**: Velocity (40pts), Seniority (20pts), ICP Fit (20pts), Content (10pts), Recency (10pts)
- **SQLite** database with 6 tables (job_postings, companies, contacts, signals, reports, excluded_customers)
- **Jinja2** HTML email reports via SMTP

## Tech Stack
- Python 3.10+, Click (CLI), pypyr (pipelines), BeautifulSoup, Playwright (Pracuj.pl), requests (NoFluffJobs)
- SQLite, Jinja2, smtplib, python-dotenv
- hatchling (build backend)

## Project Structure
```
src/hr_alerter/
├── __init__.py
├── cli.py                    # Click CLI: scrape, score, report, pipeline
├── db/
│   ├── manager.py            # 10 DB functions (get_connection, init_db, insert_jobs, etc.)
│   └── schema.sql            # 6 tables + indexes
├── scrapers/
│   ├── base.py               # BaseScraper ABC (headers, rate limiting)
│   ├── nofluff.py            # NoFluffJobs REST API scraper (PRIMARY - working)
│   ├── pracuj.py             # Pracuj.pl Playwright scraper (BLOCKED by Cloudflare)
│   └── utils.py              # normalize_company_name, detect_seniority, parse_polish_date
├── scoring/
│   ├── velocity.py           # Posting frequency score (max 40)
│   ├── seniority.py          # Role seniority score (max 20)
│   ├── icp.py                # Ideal customer profile score (max 20)
│   ├── content.py            # Job description keyword score (max 10)
│   ├── recency.py            # Post freshness score (max 10)
│   └── composite.py          # calculate_final_score -> score_result_dict
├── reporting/
│   ├── composer.py            # compose_weekly_report(conn) -> dict
│   ├── sender.py              # send_email(recipient, subject, html_body) -> bool
│   └── templates/weekly_report.html
├── steps/                     # pypyr step functions
│   ├── db_init.py, scrape_pracuj.py, scrape_nofluff.py
│   ├── normalize_companies.py, score_companies.py
│   ├── generate_report.py, send_email.py, show_summary.py
└── pipelines/
    ├── daily_scrape.yaml
    └── weekly_report.yaml
```

## Current Status (Feb 8, 2026)

### COMPLETED
- [x] All 106 tests passing (`pytest tests/ -q`)
- [x] Package installs cleanly (`pip install -e .`)
- [x] NoFluffJobs scraper working — 145 HR jobs scraped and stored
- [x] Database at `~/.hr-alerter/hr_alerter.db` has 145 jobs
- [x] Scoring engine (5 dimensions, all tested)
- [x] Email reporting module (composer + sender + template)
- [x] Click CLI with 4 commands (scrape, score, report, pipeline)
- [x] pypyr pipeline definitions created

### NOT YET DONE — Next Steps (in order)
1. **Run company normalization** — links job_postings to companies table (required before scoring)
   - Can run via: `hr-alerter pipeline daily` or the normalize step directly
2. **Test `hr-alerter score`** — score the 145 companies in the DB
3. **Test `hr-alerter report`** — generate HTML email report
4. **Test pypyr pipelines end-to-end** — `hr-alerter pipeline daily` and `weekly`
5. **Configure email** — create `.env` with SMTP_EMAIL, SMTP_PASSWORD, RECIPIENT_EMAIL
6. **Initialize git repo** — project has no git repo yet
7. **Demo prep** — Thursday Feb 13

### Known Issues
- **Pracuj.pl is blocked by Cloudflare Turnstile** — all automated methods fail (requests, curl_cffi, Playwright headless/headed). NoFluffJobs API is the primary data source for now.
- **NoFluffJobs API** (`https://nofluffjobs.com/api/posting`) returns all ~19k postings, filtered client-side for HR category + keywords → ~145 jobs

## Key Contracts
- **job_posting_dict**: source, job_url, job_title, company_name_raw, location, post_date, job_description, seniority_level, employment_type
- **score_result_dict**: company_id, final_score, lead_temperature, velocity, seniority, icp, content, recency, posting_count_7d, posting_count_30d, has_director_role, has_wellbeing_keywords, multi_city_expansion

## CLI Commands
```bash
hr-alerter scrape --source nofluff --pages 1    # Scrape jobs (nofluff/pracuj/all)
hr-alerter score                                 # Score all companies
hr-alerter report --recipient email@example.com  # Generate and send report
hr-alerter pipeline daily                        # Run full daily pipeline
hr-alerter pipeline weekly                       # Run full weekly pipeline
```

## Reference Documentation
See `docs.md` for full technical specification.
See `team-config.json` for the parallel agent build configuration.

---
> Source: [rangapin/hr-hiring-signals](https://github.com/rangapin/hr-hiring-signals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
