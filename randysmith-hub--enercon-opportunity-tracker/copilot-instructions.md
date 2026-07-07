## enercon-opportunity-tracker

> Authoritative spec: `docs/enercon-africa-opportunity-tracker-spec-v0.1.md`. Read it in full before any work. If this file and the spec conflict, stop and ask.

# CLAUDE.md - Africa Opportunity Tracker (Enercon Asia Pte Ltd)

Authoritative spec: `docs/enercon-africa-opportunity-tracker-spec-v0.1.md`. Read it in full before any work. If this file and the spec conflict, stop and ask.

## What this is
Automated pipeline capturing tenders and leading-indicator signals (financing approvals, ESIA disclosures, permits, hiring) for EPC energy integration, data center power, and off-grid/DRE opportunities across Africa, normalized into a scored opportunity register inside the existing LMS Postgres database.

## Stack
- Python 3.11, httpx, Playwright (JS portals only), APScheduler or cron
- PostgreSQL 14+, all new objects in dedicated schema `opps`, extension `pg_trgm`
- Claude API for enrichment/classification (deterministic rules first, LLM fallback, confidence < 0.7 to review_queue)
- Secrets via `.env` only: `DATABASE_URL_STAGING`, `DATABASE_URL_PROD`, `ANTHROPIC_API_KEY`, `SERPAPI_KEY`, `ALERT_WEBHOOK`. Never commit secrets. `.env` is gitignored.

## HARD RULES: change management (mandatory, no exceptions, regardless of perceived risk)
Quick changes that don't seem to need rollback are the ones most likely to need it.

For ANY database or LMS modification (schema, views, caches, dashboard code, ingestion logic):
1. Git commit + tag pre-change state before touching anything (`lms-pre-<change>-YYYYMMDD`).
2. Backup affected database objects: `CREATE TABLE backup.<object>_pre_<change> AS SELECT * FROM <object>;`
3. Save pre-change DDL for affected views/tables to `ddl_snapshots/` (pg_dump --schema-only, pg_get_viewdef).
4. Snapshot verification figures (row counts for every existing table) to `verification/` and commit.
5. Maintain `CHANGELOG.md` live during execution, one row per action (template below).
6. Document the explicit rollback procedure with verification steps BEFORE executing the change.
7. Verification gates between stages. Any failed gate = stop and roll back. Never fix forward through a failed gate without explicit user approval.

Additional standing constraints:
- All DDL lives in reviewed migration files under `migrations/`. No ad hoc DDL in psql sessions.
- All writes are confined to schema `opps` until gate G4. Touching any non-opps object at any point requires explicit user approval plus the full per-object protocol above.
- Staging first, always. Rehearse the G1 rollback on staging (`DROP SCHEMA opps CASCADE;` then re-verify counts) before any production DDL.
- Present a plan and wait for approval before executing any gate.

## Gates (condensed; full detail in spec section 6)
- G0 Pre-flight: git tag, schema-only pg_dump, verification count snapshot, changelog opened.
- G1 Schema migration: additive only, single transaction, `opps.*` + pg_trgm only. Verify zero diff on pre-existing table counts and information_schema shows only new objects. Rollback: drop schema, re-verify.
- G2 Ingestion dry run: adapters write to `opps.sources` and `opps.raw_signals` only. Verify: >= 1 signal per adapter, dup-hash rate < 5%, fetch errors < 10%, zero writes outside `opps`. Rollback: disable scheduler, truncate raw_signals.
- G3 Normalization live: zero orphan FKs, merge_queue < 15% of opportunities, 20-record manual spot-check against source URLs, sane fit_score distribution. Rollback: truncate normalized tables, raw layer preserved for replay.
- G4 LMS surfacing: first gate that may touch existing LMS objects. Full per-object backup, saved DDL, pre-written and staging-rehearsed rollback SQL, dashboard smoke tests.

## Changelog template (CHANGELOG.md)
| ts_utc | operator | gate | action | objects | verification result | rollback tested | notes |
|---|---|---|---|---|---|---|---|

## Repo layout
```
migrations/        # numbered SQL migration files, one per gate change
adapters/          # one module per source, implements fetch() and parse()
enrichment/        # rules, LLM classifier, entity resolution, scoring
ops/               # runner, scheduler, health, alerting, count scripts
verification/      # committed verification snapshots per gate
ddl_snapshots/     # pre-change DDL captures
config/            # sources.yaml, entity seed list, query bank, fit weights
docs/              # the spec
tests/             # per-adapter tests against recorded fixtures
```

## Scraper compliance (non-negotiable)
Respect robots.txt. Identified user agent with contact email. Conditional GETs. Max 1 request per 2-5 seconds per domain. No auth circumvention. No LinkedIn scraping under any framing. Retain source attribution on every record.

## Engineering conventions
- Verify every endpoint tagged `VERIFY` in the spec before writing its parser; APIs move.
- Record HTTP fixtures for each adapter; tests run offline against fixtures.
- Adapter failures increment `consecutive_failures`; three consecutive failures surfaces in `v_source_health` and the weekly digest. Fail loud, never silent.
- Stage only moves forward unless evidence indicates frozen/cancelled; every transition writes `stage_events`.

## Definition of done, v0.1
20+ active sources green in `v_source_health`; weekly digest live; at least one pursue-grade opportunity per segment manually validated against source; rollback rehearsed and documented for G1 and G4.

---
> Source: [randysmith-hub/Enercon-Opportunity-Tracker](https://github.com/randysmith-hub/Enercon-Opportunity-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
