## osrs-clan-events

> Discord bot for OSRS clan competitions: weekly Boss of the Week (BOTW) and Skill of the Week (SOTW), personal best (PB) leaderboards with proof moderation, and donation tracking.

# Agent Instructions

Discord bot for OSRS clan competitions: weekly Boss of the Week (BOTW) and Skill of the Week (SOTW), personal best (PB) leaderboards with proof moderation, and donation tracking.

**Primary IDE: Cursor.** Agent context lives in [`.cursor/rules/osrs-events.mdc`](.cursor/rules/osrs-events.mdc). This file is the portable copy for compound-engineering skills and other tools that read `AGENTS.md`.

`project_tracker: github`

## Quick Start

```bash
make run          # run locally
make test         # go test ./...
go test ./internal/database/... -run PBCategor   # focused PB category tests
```

See [README.md](README.md) for env vars and [DEPLOY.md](DEPLOY.md) for production.

## Directory Layout

```
cmd/osrs-events/       Bot entrypoint
cmd/r2-check/          R2 connectivity check
internal/              config, database, discord, firebase, osrs, proofstorage, scheduler
docs/
  solutions/           Documented solutions — search here first
  adr/                 Architecture decision records
CONCEPTS.md            Domain vocabulary (PB, BOTW/SOTW, rollover)
.cursor/rules/         Cursor agent rules (always applied)
```

## Knowledge Store

Before implementing, debugging, or designing in a documented area, search **`docs/solutions/`** (YAML frontmatter: `module`, `tags`, `problem_type`).

After solving a non-trivial problem, run **`/ce-compound`**. To audit stale docs, run **`/ce-compound-refresh`** with a scope hint.

## Domain Vocabulary

Use **[CONCEPTS.md](CONCEPTS.md)** for PB Category, Leaderboard Group, Proof Queue, Rollover, etc. Do not invent synonyms.

## Working Agreement

- **Language:** Go. Match existing package layout and naming.
- **Tests:** `go test` on packages you change. PB category seeds need `sqlite_pb_test.go` count and slice index updates.
- **PB categories:** seed in `sqlite_store.go` (`seedPBCategories`); no migration for new slugs (`ON CONFLICT DO UPDATE`).
- **Metrics:** BOTW/SOTW pools = Firebase Remote Config; PB categories = SQLite seeds.
- **Branching:** Feature branch for non-trivial work. Do not commit unless asked.

## Compound Engineering

| Skill | When |
|-------|------|
| `/ce-brainstorm` | Scope unclear |
| `/ce-plan` | Requirements ready |
| `/ce-work` | Implement |
| `/ce-compound` | Document a solved problem |
| `/ce-compound-refresh` | Audit stale solution docs |

## Key Code Paths

| Feature | Path |
|---------|------|
| PB submit / moderate | `internal/discord/cmd_submit_pb.go`, `services/pb.go` |
| PB seeds | `internal/database/sqlite_store.go` |
| Rollover | `internal/scheduler/completion.go` |
| Metric pick | `internal/discord/services/event.go` |
| Guild setup | `internal/discord/services/initializer.go` |
| Proof storage | `internal/proofstorage/r2.go` |

---
> Source: [Ainsley2007/osrs-clan-events](https://github.com/Ainsley2007/osrs-clan-events) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
