## ai-powered-running-coach

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Takbo Running Coach — Claude Reference

## Commands

```bash
# Start dev server (hot reload, http://localhost:8000)
python run.py

# Backend tests (pure-logic only, no Claude API calls)
python -m pytest tests/test_coach.py
python -m pytest tests/test_coach.py::test_fmt_pace  # single test

# Frontend tests (from frontend/tests/)
cd frontend/tests && npm test       # run once
cd frontend/tests && npm run test:watch  # watch mode
```

## Stack
- **Backend**: FastAPI (Python), Supabase (PostgreSQL), Claude API (`claude-opus-4-6`)
- **Frontend**: Vanilla JS + HTML, Flatpickr for date pickers, CSS variables (`var(--accent)`, `var(--text)`, `var(--text-sec)`)
- **Auth**: JWT + bcrypt via `backend/auth.py`; optional Google OAuth; all endpoints use `Depends(get_current_user)`
- **Email**: Resend service via `backend/email_service.py` for email verification
- **Deploy**: Docker + Fly.io (`fly.toml`, Singapore region)

## Key Files
| File | Purpose |
|---|---|
| `backend/coach.py` | All Claude AI calls: plan gen, feedback, program skeleton |
| `backend/routers/plans.py` | Training plan endpoints |
| `backend/routers/runs.py` | Run log endpoints + gamification |
| `backend/routers/integrations.py` | Strava OAuth + sync |
| `backend/routers/goals.py` | Goal CRUD |
| `backend/routers/onboarding.py` | Runner assessment + AI follow-up questions |
| `backend/routers/profile.py` | User profile updates |
| `backend/routers/progress.py` | Aggregate stats + monthly summaries |
| `backend/routers/gamification.py` | XP, level, streaks, achievements fetch |
| `backend/schemas.py` | All Pydantic models |
| `migrations/supabase_schema.sql` | Full DB schema (run in Supabase SQL editor) |

Additional migrations in `migrations/`: `supabase_migration_profile.sql`, `supabase_migration_integrations.sql`, `supabase_migration_personal_bests.sql`, `supabase_migration_rls_fix.sql` — apply in order if setting up fresh.

## Database Tables
- **users** — id, email, name, password_hash, weight_kg, max_hr, age, height_cm, birthdate, avatar_url, google_id
- **run_logs** — id, user_id, date (local date, not UTC), distance_km, duration_min, pace_per_km, heart_rate_avg, effort_level, notes, ai_feedback, strava_activity_id, route_polyline, elevation_gain, sport_type, coach_note
- **goals** — id, user_id, race_type, race_date, target_time_min, goal_type ('race'|'fitness'|'weight_loss'|'pb_attempt'|'custom'), goal_description, target_value, target_unit, target_weight_kg
- **training_programs** — id, user_id, goal_id, total_weeks, start_date, end_date, skeleton_json, status ('active'|'completed'|'abandoned')
- **training_plans** — id, user_id, week_start, plan_json, generated_at, program_id (FK → training_programs), week_number
- **runner_assessments** — user_id, experience_level, weekly_runs, weekly_km, primary_goal, available_days, load_capacity, injury_history, medications, ai_followup_q/a
- **strava_tokens** — user_id, access_token, refresh_token, expires_at, athlete_id
- **user_gamification** — user_id, xp, level, current_streak, longest_streak, total_runs, total_km
- **achievements** — id, user_id, achievement_key, unlocked_at
- **athlete_summaries** — user_id, summary_text, generated_at, runs_included
- **user_personal_bests** — id, user_id, race_type, time_min, date_achieved, source ('manual'|'strava')
- **user_race_predictions** — user_id, predictions_json, generated_at

## Training Program System
- A **training_program** is the parent: full-duration periodization skeleton from today → goal end date
- Each **training_plan** is a weekly child, linked via `program_id` + `week_number`
- `skeleton_json`: JSON array of `{week_number, phase, focus, target_km, target_long_run_km, key_workout, notes}`
- Standard phases — race goals: Base Building → Build → Peak → Taper; non-race: Foundation → Progression → Consolidation
- Week generation is **on-demand** (not all at once): POST `/api/plans/next-week` uses the skeleton entry for that week as context
- Only one program is `active` at a time; creating a new one marks previous as `abandoned`

## Critical Rules
- **Strava dates**: always use `start_date_local` (not `start_date`) to avoid UTC→local date shift
- **Plan dates**: compare as `YYYY-MM-DD` strings, never parse as `Date()` without specifying local components, to avoid timezone shifts
- **Week start**: `_current_monday()` in `plans.py` returns the current UTC Monday; client may pass `local_monday` to override for non-UTC timezones
- **Backwards compatibility**: `program_id` and `week_number` are nullable on `training_plans` — legacy weekly plans (no program) must keep working
- **Idempotency**: generating next-week checks for existing plan with same `program_id + week_number` before calling Claude

## AI Coaching Functions (`backend/coach.py`)
| Function | Purpose |
|---|---|
| `generate_program_skeleton(goal, total_weeks, assessment, user_name, personal_bests)` | Multi-week periodization outline |
| `generate_weekly_plan(..., week_context)` | 7-day detailed plan; `week_context` from skeleton guides Claude |
| `generate_run_feedback(run, recent_runs, ...)` | Per-run coaching feedback |
| `should_adjust_plan(run, recent_runs, ...)` | Returns `{adjust: bool, reason: str}` |
| `adjust_upcoming_workouts(...)` | Modifies remaining week's workouts after a plan adjustment |
| `generate_workout_variation(day, ...)` | Alternative workout at same intensity |
| `generate_athlete_summary(...)` | Rolling athlete context summary; runs as background task after each run |
| `get_onboarding_followup(...)` | Follow-up questions during runner assessment |
| `predict_race_times(...)` | Race time predictions; results cached in `user_race_predictions` |

## Frontend Patterns
- API calls via `api.get/post/patch/delete` in `frontend/js/api.js`
- Auth check: `requireAuth()` at page load; user data from `getUser()`
- Modal pattern: backdrop div with `hidden` class, `onclick="closeModal(event)"` on backdrop, check `e.target === backdrop` inside handler
- Loading state: show `#plan-loading` div, hide `#plan-container`, then swap after response
- `escapeHtml(str)` must be used for any user-generated content rendered as innerHTML

---
> Source: [errolagatep/AI-Powered-Running-Coach](https://github.com/errolagatep/AI-Powered-Running-Coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
