## bracketsball

> March Madness bracket challenge app where users compete in pools by predicting NCAA tournament game outcomes.

# Bracketsball

March Madness bracket challenge app where users compete in pools by predicting NCAA tournament game outcomes.

## Tech Stack

Next.js (TypeScript), Postgres (Neon), ShadCN + Tailwind, React Hook Form, Better Auth (Google/Discord), Vercel hosting, cron-job.org for scheduled syncs.

## Key Documentation

- `docs/business/originalVision.md` — Product vision, feature descriptions, scoring rules, and full feature table with MVP flags. Read this to understand what the app does and the domain model (pools, brackets, entries, scoring, tiebreakers).
- `docs/business/backlog.md` — Prioritized user stories organized by epic (Auth, Profile, Account, Pools Setup, Sports Data, Pool Members, Bracket Creation, Bracket Visibility). 13 MVP stories, 4 post-MVP. Read this before starting any feature work to understand acceptance criteria.
- `docs/technical/stack.md` — Tech stack summary.
- `docs/technical/standards.md` — Codebase standards (WIP).
- `docs/technical/ways-of-working.md` — How Claude Code should operate: task tracking, decision logging, review process, testing, and feedback loops. **Read this before starting any task.**

## Claude Code Skills

- **`/pre-review`** — Run this before marking any task `ready-for-review`. It automates self-code review, format/lint/build checks, acceptance criteria verification, and task file updates. See `docs/technical/ways-of-working.md` section 3.
- **`/review-pr`** — Use for automated PR code reviews.
- **Auto-format hook** — Prettier runs automatically on every file edit (configured in `.claude/settings.json`).

## Domain Concepts

- **Pool**: A group competition with configurable scoring, max brackets per user (1-10), and max participants (2-100). Has leaders and members.
- **Bracket Entry**: A user's set of tournament picks within a pool. Auto-saves. Requires all games picked + tiebreaker (championship total score) to submit.
- **Scoring**: Points per round — First Four: 0, R64: 1, R32: 2, Sweet 16: 4, Elite 8: 8, Final Four: 16, Championship: 32.
- **Standings**: Ordered by points desc, potential points desc, then alphabetical.
- **Sports Data**: Synced from ESPN API on a cron schedule.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paul-macfarlane)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/paul-macfarlane)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
