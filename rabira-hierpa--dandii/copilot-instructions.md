## dandii

> Monorepo: `web/` is the Next.js app (see `web/CLAUDE.md`), `data/gtfs-2026/`

# Dandii — Addis Ababa Transit

Monorepo: `web/` is the Next.js app (see `web/CLAUDE.md`), `data/gtfs-2026/`
holds the vendored DT4A GTFS feed, `otp-data/` feeds the OpenTripPlanner
container, `docker-compose.yml` runs postgis + otp + web for Coolify.

## Skill routing

When the user's request matches an available skill, invoke it via the Skill tool. When in doubt, invoke the skill.

Key routing rules:
- Product ideas/brainstorming → invoke /office-hours
- Strategy/scope → invoke /plan-ceo-review
- Architecture → invoke /plan-eng-review
- Design system/plan review → invoke /design-consultation or /plan-design-review
- Full review pipeline → invoke /autoplan
- Bugs/errors → invoke /investigate
- QA/testing site behavior → invoke /qa or /qa-only
- Code review/diff check → invoke /review
- Visual polish → invoke /design-review
- Ship/deploy/PR → invoke /ship or /land-and-deploy
- Save progress → invoke /context-save
- Resume context → invoke /context-restore
- Author a backlog-ready spec/issue → invoke /spec

---
> Source: [rabira-hierpa/dandii](https://github.com/rabira-hierpa/dandii) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
