## newsletter-aggregator

> You are working on the **Newsletter Aggregator** project. This file loads into every session

# CLAUDE.md — read this first (auto-loaded every session)

You are working on the **Newsletter Aggregator** project. This file loads into every session
(any teammate, cloud or local), so it's the shared starting point.

## Get oriented before doing anything
**Read [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md)** — it's the full state, every decision, and a
**Session Log** of what past sessions did. That's how you "continue" the shared thread of work.
Live per-newsletter status is in `data/sources.csv`. Design is in `PHASE4_DESIGN.md`.

## Hard rules
- **Never commit secrets.** Gmail OAuth token/credentials are gitignored and machine-local.
  `~/.config/google-skill/credentials.json` and `.claude/*.local.json` must never be pushed.
- **Gmail access is LOCAL-only.** Only a session with the authenticated `tools/google-skill` tool
  can read `notifyy1008@gmail.com`. Cloud/teammate sessions usually can't — don't wait on Gmail there.
  Do Gmail work in the local session, push results (e.g. `data/sources.csv`), teammates `git pull`.
- **No Claude/AI in the capture pipeline.** Phase 5 enrichment uses Groq or local Qwen — never Claude.

## How we collaborate (see CONTRIBUTING.md for detail)
- Work on a **branch**, push, open a **PR** — don't commit directly to `main`. The owner reviews PRs.
- When a session did meaningful work, **append a note to `PROJECT_CONTEXT.md`** (Session Log) and
  include it in your PR, so everyone sees what was discussed/decided.

## Where things stand (2026-08-07)
Phases 1–3 done (~35 newsletters confirmed delivering; see `sources.csv`). Phase 4 (archive:
SQLite + .eml + .md) designed + proven on one email. Phase 5 (LLM enrichment) designed. Next:
build the capture script + backfill, then the digest. Full detail in `PROJECT_CONTEXT.md`.

---
> Source: [KavyaJain321/newsletter-aggregator](https://github.com/KavyaJain321/newsletter-aggregator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
