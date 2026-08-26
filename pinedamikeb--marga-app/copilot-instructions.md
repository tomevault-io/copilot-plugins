## marga-app

> Read first for any Codex/agent work in this repo:

# MARGA Agent Rules

Read first for any Codex/agent work in this repo:
1. `/Volumes/Wotg Drive Mike/GitHub/Marga-App/HANDOFF.md`
2. `/Volumes/Wotg Drive Mike/GitHub/Marga-App/MASTERPLAN.md`
3. `/Volumes/Wotg Drive Mike/GitHub/Marga-App/AGENTS.md`

For database migration, backend cutover, rescue sync, Margabase compatibility APIs, or production write paths, also read:
`/Volumes/Wotg Drive Mike/GitHub/marga-platform/skills/marga-database-migration/SKILL.md`

## Core Purpose
Codex is responsible for protecting the owner from unnecessary cost and repeated work.

Before doing a task, choose the cheapest safe path that preserves business truth. Avoid broad paid reads/writes, unnecessary SaaS services, repeated Firebase/API scans, duplicate manual work, and recurring costs that can be eliminated with local scripts, cached reports, Postgres views, or reusable tooling.

## Reuse And Memory
- Check `HANDOFF.md`, `MASTERPLAN.md`, existing scripts, reports, and previous utilities before re-solving a problem.
- If a solution will be needed again, save it as documentation, a script, an automation, or a skill.
- Keep reusable MARGA skills under `/Volumes/Wotg Drive Mike/GitHub/marga-platform/skills`; link broadly useful skills into `/Users/mike/.codex/skills` so future sessions can discover them.
- Treat every real production mistake or repeated prompt as a learning event. After fixing it, actively decide whether the resolution should become a reusable skill, script, checklist, or doc rule so future Codex sessions prevent the same class of error.
- If a command or report worked once for a real incident, make it repeatable and record when to use it.
- Prefer local backups and saved reports before live paid backend reads.

## Product Design Guardrails
- Anticipate user and staff mistakes before they happen.
- Use searchable dropdowns for real entities such as customers, invoices, branches, machines, employees, OR numbers, and accounts.
- Keep labels minimalist: if a field is already clearly labeled `Customer`, `Branch`, `Invoice`, or similar, do not add a second explanatory line that only repeats the same meaning. Default to clean headings and plain controls before adding helper copy.
- Use tables/grids for multi-line financial or operational data instead of free-text remarks when accuracy matters.
- Add validation and audit trails for billing, collections, payments, schedules, petty cash, DR/releasing, and status changes.
- Prefer reusable helpers and database-side rules over copy-paste browser logic that can drift between modules.

## Deploy And Release (Canonical)
- Work in `/Volumes/Wotg Drive Mike/GitHub/Marga-App` on `main` for production staff testing at `app.marga.biz`.
- After updating local `Marga-App`, automatically deploy that updated code to the live app served at `app.marga.biz` so the owner can test immediately. Do not stop to ask whether to deploy; deploy automatically unless the owner explicitly says not to.
- Wait for the owner's live `app.marga.biz` result first, then sync to `Marga-App-staging` / `codex/staging`, then commit and push the verified change to GitHub `main` unless the owner explicitly requests a different order.
- Verify on `app.marga.biz` with a hard refresh for service worker changes.
- Bump `service-worker.js` cache name and critical script `?v=` query strings on every deploy.

## Migration Guardrail
Migration is not complete when data is copied. Migration is complete only when old backend secrets/config are removed, old domains are blocked, service worker cache is reset, all write paths are proven against the new database through the production URL staff use, and stale writes from the old database are reconciled with an auditable report.

For migrated modules, prove writes as well as reads. Create/update/delete paths, numeric ID allocation, invoice/OR/DR uniqueness, schedule creation, release-item creation, payment posting, petty cash voucher lines, and audit rows must be checked through the same production Margabase route staff use.

## Default Question
For every change, ask:
- What can create cost?
- What can create repeated manual work?
- What can break data accuracy?
- What can staff type incorrectly?
- What should be saved so this does not need to be prompted again?

---
> Source: [PinedaMikeB/marga-app](https://github.com/PinedaMikeB/marga-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
