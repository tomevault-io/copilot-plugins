## jobpilot

> Personal job-application pipeline: ATS board diffing → filters → Telegram digest → human-approved applications. Single user, laptop-first, SQLite.

# JobPilot

Personal job-application pipeline: ATS board diffing → filters → Telegram digest → human-approved applications. Single user, laptop-first, SQLite.

## Plan of record

The approved design doc lives OUTSIDE this repo (personal content):
`~/.gstack/projects/JobPilot/atreus-unknown-design-20260719-180727.md`

Precedence on any conflict: **CEO amendments (C1–C11) > eng amendments (E1–E13) > design-doc deltas > PROJECT.md**. PROJECT.md §2's architecture diagram is stale (shows pgvector, IMAP, separate crons — all removed); trust the design doc.

Task lists: `~/.gstack/projects/JobPilot/tasks-eng-review-20260719-201500.jsonl` and `tasks-ceo-review-20260719-214814.jsonl`. Test plan: `atreus-unknown-eng-review-test-plan-20260719-195527.md`.

## Hard rules (never violate)

- `*.db*`, `.env`, `data/`, `resume/base.yaml` never enter git history — the DB carries third-party PII (LinkedIn contacts).
- Resume tailoring never fabricates: no skill, employer, date, or metric absent from the base YAML. Reorder/rephrase only.
- Submission is always human-click-Submit. No unattended auto-apply.
- Adapters are dumb (`PollResult`: fetch+parse+normalize only); the pipeline core alone owns upsert, first_seen seeding, status flips, dedupe (E5).
- Status flips to closed only after 2 consecutive **successful** polls with the job absent. Failed polls never flip status.
- No intelligence code (scoring/drafting) until the weekend-1 gate passes: ≥8 👍-annotated roles/week from digests, reported per market × source tier.

## Stack

Python 3.12 (pinned via uv; system 3.14 untested with deps), SQLAlchemy + Alembic on SQLite WAL, httpx, python-telegram-bot (persistent daemon for callbacks), python-jobspy, rapidfuzz. One chained pipeline run every 30 min (flock, per-source timeouts); JobSpy 3×/day and HN 1×/day fire on wall-clock windows inside that run.

## Skill routing

When the user's request matches an available skill, invoke it via the Skill tool. When in doubt, invoke the skill.

Key routing rules:
- Product ideas/brainstorming → invoke /office-hours
- Strategy/scope → invoke /plan-ceo-review
- Architecture → invoke /plan-eng-review
- Bugs/errors → invoke /investigate
- QA/testing behavior → invoke /qa or /qa-only
- Code review/diff check → invoke /review
- Ship/deploy/PR → invoke /ship
- Save/resume context → invoke /context-save, /context-restore
- Author a backlog-ready spec/issue → invoke /spec

---
> Source: [aadidotin/JobPilot](https://github.com/aadidotin/JobPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
