## brag-doc

> A self-contained record of work accomplishments, plus the Claude Code skills that maintain it.

# brag-doc

A self-contained record of work accomplishments, plus the Claude Code skills that maintain it.
Used to build promotion cases and annual-review material.

## What lives where
- `entries/YYYY/MM/YYYY-MM-DD.md` — daily journal entries (the source of truth).
- `sources/` — raw inputs like past reviews and manager feedback. The sample shipped here is
  fictional; your real inputs go here and are sensitive.
- `reviews/` — generated review drafts (gitignored).
- `.claude/skills/daily-eval/` — captures entries.
- `.claude/skills/review-builder/` — synthesizes entries into review material.

## Naming conventions
- Name each review cycle's files by the **year being reviewed**, not the year it was conducted.
  Example: a 2025 annual review (done in early 2026) is `sources/2025-annual-review.md` and, if you
  have one, `sources/2025_perf_review_template_questions.pdf`.
- Keep each cycle's template PDF alongside its review; add new ones as dated files rather than
  overwriting, so the history of how the format evolved is preserved.

## Rules
- **Git workflow.** Show me the diff before committing so I can eyeball the changes first; once
  I've looked, you can commit. Ask before pushing.
- **Entries are append-only.** Never overwrite an existing day's entry; add to it.
- **Commit identity.** Use whatever git identity is configured locally. If you don't want a personal
  address in history, set a GitHub noreply email once:
  ```
  git config user.email "YOUR_ID+YOUR_USERNAME@users.noreply.github.com"
  git config user.name "YOUR_USERNAME"
  ```
- **Privacy.** This template is public, but real `entries/` and `sources/` contain real names and
  manager feedback. Do your real journaling in a private repo (see README), and never commit real
  entry or source content to a public repo.

---
> Source: [deeheber/brag-doc](https://github.com/deeheber/brag-doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
