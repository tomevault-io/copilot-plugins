## newtonmacurriculumtester

> - Run Python through **`uv`**, not the interpreter directly: `uv run pytest tests/`, `uv run python -m tools.simulate_ramp`, etc. (Bare `python` is not on PATH — the Windows Store stub intercepts it — and we standardize on uv for env consistency.)

# Notes for Claude

## Environment

- Run Python through **`uv`**, not the interpreter directly: `uv run pytest tests/`, `uv run python -m tools.simulate_ramp`, etc. (Bare `python` is not on PATH — the Windows Store stub intercepts it — and we standardize on uv for env consistency.)
- There is no sqlite3 CLI installed; query the DB through Python's `sqlite3` module.

## Production / deploying

The app is live at **https://mathkids.seidmann.workers.dev** (Python Worker + D1, cutover
2026-07-18), behind Cloudflare Access (email allow-list + One-time PIN, managed in the Zero
Trust dashboard — not configurable via wrangler).

- **Deploy:** `uv run pywrangler deploy` (~30 s). Deploys are manual — pushing to GitHub does
  **not** update the site. Docs-only changes don't need a deploy.
- **Local Workers dev:** `uv run pywrangler dev` (localhost:8787, local D1). The plain local
  app is `uv run mathkids` (uvicorn + `mathkids.db`).
- Deploys never touch D1 data. Wrangler auth: one-time `npx wrangler login` per machine.

## Database

Two backends behind the same async helpers in `db.py` (all take a `SqliteDB`/`D1DB` adapter
as their first argument): local dev/tests use the SQLite file `mathkids.db` at the repo root;
the deployed app (Cloudflare Workers) uses D1. Schema source of truth is
`migrations/0001_init.sql`. Query the deployed data with
`npx wrangler d1 execute mathkids --remote --command "..." --json`.

Since the Cloudflare cutover (2026-07-18), **remote D1 is the system of record** — the local
`mathkids.db` is a stale pre-migration snapshot kept for dev, and `.wrangler/state` local D1
is throwaway test data (never export from it).

Tables:
- `kid` — id, name, grade, emoji, daily_goal. (Jacob = grade 2, Samuel = grade 4.)
- `session` — one row per practice session: kid_id, plan, answered, num_correct, day, started_at/ended_at.
- `attempt` — one row per question answered: kid_id, skill_id, session_id, level, prompt, expected, given, correct (0/1), response_ms, day, created_at. The full prompt text and the kid's literal answer are stored, so you can reconstruct exactly what they saw and typed.
- `skill_state` — per (kid, skill) mastery tracking: score, level, Leitner box, consec_correct, recent history, due_at, lesson_seen, introduced_at/mastered_at/last_seen_at.

The go-to query for "what did the kids miss":

```sql
SELECT k.name, a.skill_id, a.level, a.prompt, a.expected, a.given, a.created_at
FROM attempt a JOIN kid k ON k.id = a.kid_id
WHERE a.correct = 0 ORDER BY a.created_at;
```

`skill_id` values are Common Core ids (e.g. `2.NBT.A.1`); the generator for a skill lives in `src/mathkids/engine/grade{N}/{domain}.py` and is findable by grepping for the id.

## Gotchas

- Problems are regenerated deterministically from (skill, level, seed) at grading time. If you change a generator while the app is running, restart it (locally) or redeploy (Workers) before a kid answers an in-flight question, or the regenerated problem won't match what was shown.
- Problem figures are client-side SVG: generators put a JSON spec in `payload["image"]`; `public/static/images.js` renders it. Adding an image kind means adding a renderer there (Pillow is gone).
- Templates are baked into `src/mathkids/_templates_embedded.py` for the Workers bundle by `tools/embed_templates.py` (wrangler runs it automatically on dev/deploy; the file is gitignored). Local uvicorn reads the live template files, so only Workers deploys depend on it.
- "Today" (`db.today_ordinal()`) is America/New_York, not the machine/UTC date.
- Questions must require only a single answer in one input box (or one multiple-choice pick). No "list all the factors" / "next three numbers" formats — the kids find multi-answer entry confusing. `SequenceAnswer`/`SetAnswer` still exist in `answers.py` but no skill should use them.
- Prompt wording should be concrete and kid-friendly: prefer story problems with names/objects over abstract phrasing ("A number is 4 times as many as 2"), and plain words over math-register ones ("how much is the digit worth" rather than "the value of the digit").

---
> Source: [seidleroni/NewtonMACurriculumTester](https://github.com/seidleroni/NewtonMACurriculumTester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
