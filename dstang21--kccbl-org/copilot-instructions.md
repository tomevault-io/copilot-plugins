## kccbl-org

> You are working on a Laravel rebuild of the KCCBL baseball league website.

# KCCBL Copilot Instructions

You are working on a Laravel rebuild of the KCCBL baseball league website.

## Before You Edit
- Read `AGENTS.md`.
- Read `docs/implementation-status.md`.
- Read `docs/architecture.md` if the task touches models, routing, season resolution, or entity relationships.

## Project Truths
- The SQL dump `u596677651_kccbl_remake (3).sql` is the canonical schema.
- `clubs` is the public franchise entity.
- `season_teams` is the primary season-scoped standings/team record.
- `teams` is legacy compatibility data, not the public team identity.
- `seasons.active` drives default season resolution.
- Keep public URLs intact.
- Prefer Blade, controllers, models, middleware, and policies over SPA patterns.

## What Exists Already
- Public route skeleton and first public implementation slice are in place.
- Shared public layout and sports-site styling exist.
- Public pages already cover home, teams, schedule/results, standings, players, stats, game pages, playoffs, news, sponsors, parks, awards, media, scouting videos, contact, legal pages, and compatibility endpoints.
- Tests currently cover route parity and legacy route key configuration.

## Immediate Priorities
- Finish mapping the remaining schema into explicit models and relationships.
- Implement auth and profile management.
- Build player claim workflow.
- Build `/coach` and `/admin` with real authorization.
- Keep deepening public pages rather than replacing them with placeholders.

## Quality Expectations
- Do not make the site look like a generic dashboard or startup template.
- Preserve baseball-site density, hierarchy, branding, and schedule/stat readability.
- Favor real database-backed rendering over fake content.
- Maintain accessibility and responsive behavior.

## Required Validation
After meaningful changes, run the most relevant commands:
- `php artisan route:list`
- `php artisan test`
- `npm run build`

## Required Documentation Updates
If you materially change the app, update:
- `docs/implementation-status.md`
- `docs/architecture.md`

Update `AGENTS.md` when the working rules, priorities, or repo truths change.

---
> Source: [dstang21/kccbl_org](https://github.com/dstang21/kccbl_org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
