## osrs-events

> OSRS Events bot — project context, domain vocabulary, compound engineering


# OSRS Events (Cursor)

Go Discord bot: BOTW/SOTW weekly competitions, PB leaderboards with proof moderation, donations.

## Before you change code

1. **Domain terms** — [CONCEPTS.md](CONCEPTS.md) for PB Category, Leaderboard Group, Proof Queue, Rollover, etc.
2. **Past learnings** — search [docs/solutions/](docs/solutions/) (`module`, `tags`, `problem_type` in frontmatter).
3. **Full orientation** — [AGENTS.md](AGENTS.md) for layout, code paths, compound workflow.

## Compound engineering

| Skill | When |
|-------|------|
| `/ce-work` | Implement |
| `/ce-compound` | Document a solved problem in `docs/solutions/` |
| `/ce-compound-refresh` | Audit stale solution docs |

Use `/ce-brainstorm` and `/ce-plan` when scope needs framing first.

## Conventions

- **Tests:** `make test` on packages you touch.
- **PB categories:** `internal/database/sqlite_store.go` (`seedPBCategories`); update `sqlite_pb_test.go` counts and slice indices.
- **Scope:** Minimal diffs. Do not commit unless asked.

`project_tracker: github`

---
> Source: [Ainsley2007/osrs-clan-events](https://github.com/Ainsley2007/osrs-clan-events) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
