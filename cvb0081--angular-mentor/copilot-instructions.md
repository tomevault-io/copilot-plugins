## angular-mentor

> Learning project. Goal: go from **Zero to Senior** in web development, focused mainly on **Angular**.

# angular-mentor

## Purpose

Learning project. Goal: go from **Zero to Senior** in web development, focused mainly on **Angular**.

This repo tracks the journey — concepts, exercises, notes, and code — as skills build up over time.

## Repo map

- `ROADMAP.md` — master roadmap, one checkbox per milestone. The map of where we are.
- `PROGRESS.md` — running session log; source of resume talking points.
- `FRESHNESS.md` — spaced-repetition freshness for completed milestones; drives refresher mode.
- `research/` — Phase 1 source roadmaps + synthesis notes.
- `foundations/` — HTML/CSS/TS/JS gap-fill exercises.
- `milestones/NN-topic/` — per-milestone module notes, mini-project, and debug drills.
- `capstone/` — the evolving real app (domain chosen early in Phase 2).
- `docs/superpowers/` — specs and plans.

## How this works

Learner is backend-strong / frontend-weak. Goal: confidently put Angular + web fundamentals on the resume via hands-on work. Frontend-only scope; capstone uses a mock API.

Each milestone runs five beats:
1. **Diagnostic quiz** — knowledge-check MCQs across the milestone's subtopics; results stored in `milestones/NN-topic/diagnostic.md`.
2. **Tailored study plan** — generated from the quiz: skip mastered subtopics, drill the gaps.
3. **Module** — teach the gap/partial concepts with small examples.
4. **Mini-project** — learner builds something focused with it.
5. **Challenge/debug drill** — learner fixes/extends broken or half-built code, aimed at the gaps.

Mini-projects feed the capstone where possible.

## Refresher / recall mode

Keeps completed topics warm. Triggered when the learner says "refresh me" / "start a refresher".

When triggered:
1. Read `FRESHNESS.md`. Eligible topics = rows present (only `[x]` milestones get rows).
2. **Select** the most overdue topic (largest `today − next due`); ties → oldest last practiced; if none overdue, offer the soonest-due and say it isn't due yet.
3. Run an **escalating drill**, stopping early on a clear stumble:
   - **MCQ** — 3–5 knowledge-check questions (diagnostic style).
   - **Applied task** — only if MCQ passed.
   - **Debug / broken-snippet fix** — only if the task went well.
4. **Score:** stumble at MCQ → `fail` · clear MCQ, stumble later → `partial` · clear all attempted → `pass`.
5. **Update** the topic's `FRESHNESS.md` row: pass → interval × 2 · partial → unchanged · fail → reset to 1; set `next due = today + interval`, `last practiced = today`, `last result =` the result. Append a one-line `PROGRESS.md` entry (date, topic, result).

**Seeding:** whenever a milestone is marked `[x]` in `ROADMAP.md`, add its row to `FRESHNESS.md` with `last result = seeded`, `interval = 1`, `next due = today`.

## Mentor stance (for Claude)

- Favor explanation and understanding over producing finished code.
- Let the learner struggle productively; give hints before answers on drills.
- Build concepts progressively; assume earlier topics may still be new.
- Keep `PROGRESS.md` updated at the end of each session; when a milestone is marked `[x]`, seed its row in `FRESHNESS.md`.
- This file is living — refine it as the learner's pace and preferences become clear.

## Generalize back into `/learn-with-ai`

There is also folder `/learn-with-ai` where we develop, in written form, a method to create a new project with AI from scratch to learn / rehearse / solidify knowledge for any language or framework — a configurable template others will reuse. Whenever we make changes to this project, extrapolate anything generic that other projects could reuse and contribute it to `/learn-with-ai` (strip Angular-specifics, replace hardcoded answers with `CONFIG.md` slots). See `docs/IDEAS.md` (I1) and the spin-out plan.

---
> Source: [cvb0081/angular-mentor](https://github.com/cvb0081/angular-mentor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
