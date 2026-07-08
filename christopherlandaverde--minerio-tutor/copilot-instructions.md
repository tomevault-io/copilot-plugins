## minerio-tutor

> Sabia is a **GUI desktop app** (Tauri + SvelteKit) for learning Mineiro Portuguese (Brazilian Portuguese, Minas Gerais dialect). All learning happens through the app's UI — clicking buttons, typing in input fields, navigating pages. This is NOT a CLI tool.

# Sabiá — Mineiro Portuguese Learning App

Sabia is a **GUI desktop app** (Tauri + SvelteKit) for learning Mineiro Portuguese (Brazilian Portuguese, Minas Gerais dialect). All learning happens through the app's UI — clicking buttons, typing in input fields, navigating pages. This is NOT a CLI tool.

See `LEARNING_SYSTEM.md` for the learning methodology, algorithms, and content design.

## Key Files

| File | Purpose |
|------|---------|
| `fluent-mineiro/src-tauri/src/lib.rs` | Rust backend: DB migrations, Tauri plugins |
| `fluent-mineiro/src/lib/db.ts` | SQLite helpers (profile, attempts, SRS, sessions) |
| `fluent-mineiro/src/lib/exercises.ts` | Exercise scoring + SRS update |
| `fluent-mineiro/src/lib/adaptive.ts` | CEFR adaptive difficulty |
| `fluent-mineiro/src/lib/sm2.ts` | SM-2 spaced repetition algorithm |
| `fluent-mineiro/src/lib/session-planner.ts` | Smart daily session assembly (legacy; kept for reference) |
| `fluent-mineiro/src/lib/lesson.ts` | Lesson Arc assembly (Teach→Recognize→Produce→Capstone) + pluggable `chooseTheme` seam |
| `fluent-mineiro/src/lib/emoji-map.ts` | Emoji lookup for concrete vocab (render-time, no schema change) |
| `fluent-mineiro/src/lib/achievements.ts` | 16 achievement badges + check logic |
| `fluent-mineiro/src/lib/challenges.ts` | Weekly challenge generation + tracking |
| `fluent-mineiro/src/lib/claude.ts` | Claude API (conversation + coaching notes) |
| `fluent-mineiro/src/lib/content.ts` | Exercise content loader (A2/B1/B2) |
| `fluent-mineiro/src/lib/components/ExercisePlayer.svelte` | Shared exercise UI (all 6 types) |
| `fluent-mineiro/src/routes/` | SvelteKit pages (dashboard, session, lesson, review, conversation, writing, reading, progress, achievements, settings). `/session` plays a themed Lesson Arc (Teach→Recognize→Produce→Capstone) assembled by `lesson.ts`, with SRS reviews as a warm-up. |
| `LEARNING_SYSTEM.md` | Complete methodology guide |

## SQLite Tables

| Table | Purpose |
|-------|---------|
| `profile` | Key-value store (streak, XP, level, daily goal, API key, settings) |
| `attempts` | Every exercise answer (exercise_id, correct, quality, mistake_type) |
| `srs_state` | SM-2 state per exercise (easiness, interval, next_review) |
| `sessions` | Session start/end, exercises completed, XP earned |
| `achievement_unlocks` | Which badges are unlocked + when |
| `weekly_challenges` | Active challenges, progress, completion status |

## Mineiro Dialect Notes

The app teaches standard Brazilian Portuguese WITH Mineiro flavor:
- **uai** — universal interjection (surprise, agreement, filler)
- **trem** — "thing" (replaces coisa/negocio in casual speech)
- **ce** — contracted "voce"; **oce** in some rural areas
- **so** — vocative particle, like "cara" (O so, ce viu?)
- **no!** — exclamation (short for "Nossa Senhora!")
- **bao** — "bom" (Ce ta bao?)
- **Dropped gerund 'd'**: falano, comeno, fazeno (not falando, comendo, fazendo)
- Tendency to shorten words and softer, melodic intonation

## Learning Principles

1. **Active Recall** — ask before showing answers
2. **Spaced Repetition (SM-2)** — intervals based on performance
3. **Immediate Feedback** — correct with clear explanations
4. **Interleaving** — mix topics per session
5. **Comprehensible Input (i+1)** — slightly above current level
6. **Desirable Difficulty** — target 60-70% success rate

## App Architecture

- **Frontend:** SvelteKit (SSG) with Svelte components
- **Backend:** Tauri (Rust) with SQLite via `tauri-plugin-sql`
- **AI:** Claude API for conversation, coaching, and adaptive content
- **Data:** All learner data in SQLite (`user.db`), no flat JSON files

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.

## gstack

Use /browse from gstack for all web browsing. Never use mcp__claude-in-chrome__* tools.
Available skills: /office-hours, /plan-ceo-review, /plan-eng-review, /plan-design-review,
/design-consultation, /review, /ship, /land-and-deploy, /canary, /benchmark, /browse,
/qa, /qa-only, /design-review, /setup-browser-cookies, /setup-deploy, /retro,
/investigate, /document-release, /codex, /careful, /freeze, /guard, /unfreeze,
/gstack-upgrade.

If gstack skills aren't working, run `cd .claude/skills/gstack && ./setup` to build the binary and register skills.

---
> Source: [ChristopherLandaverde/minerio_tutor](https://github.com/ChristopherLandaverde/minerio_tutor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
