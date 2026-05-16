## coachify

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Bot

```bash
source .venv/bin/activate
python main.py
```

Required environment variables in `.env`:
- `TELEGRAM_BOT_TOKEN` — required
- `OPENAI_API_KEY` — optional; AI plan generation silently skipped if absent
- `SUPABASE_URL` — required
- `SUPABASE_KEY` — required

---

## Standard Operating Procedure (SOP)

### Workflow Stages

1. **Planning** — User provides a planning file (from Obsidian). Analyze it, suggest improvements, and wait for approval before touching code.
2. **Context Reset** — After task agreement, user runs `/clear` to save tokens. Start fresh with clean context.
3. **Execution** — User provides the first task and enters `/plan` mode. Use superpowers + LSP for discovery before reading files. Ask clarifying questions before starting.
4. **Review** — After coding, generate a diff/summary for PR review. Do not mark a task "Done" until "Review Approved" is received.
5. **Iteration** — Fix code based on review feedback (from user or external AI like Gemini) before production.

### Behavior Rules

- **Be extremely concise. No yapping.**
- Always ask clarifying questions before starting a task in plan mode.
- Always use superpowers skills and LSP for discovery — minimize cold full-file reads.
- SQL files are core logic — analyze them alongside Python files for any DB-related task.
- A task is only "Done" after explicit "Review Approved" from the user.

---

## Architecture Overview

This is a Telegram fitness/nutrition coaching bot (Hebrew + English). The codebase is **fully modular** — there is no monolith `coachify_bot.py`.

### Module Map

| File | Purpose |
|---|---|
| `main.py` | Entry point — ConversationHandler wiring + polling retry loop |
| `config.py` | Env vars, ALL state constants, default schedules, TOS buttons, RTL Unicode |
| `bot_helpers.py` | Shared utils: `get_message`, `user_lang`, `kb`, `parse_number_robust`, `send_long_reply`, `check_rate_limit`, `safe_parse_time`, etc. |
| `onboarding.py` | Profile onboarding flow: start → TOS → lang → age/height/weight/gender/fitness/diet/goal → confirm → schedule |
| `plan_handlers.py` | Plan generation/display: `show_or_build_plan`, `generate_and_send_plan`, `plan_cmd`, `replan_cmd`, `showplan_cmd`, `show_meals_cmd`, `show_workouts_cmd` |
| `reminder_handlers.py` | Reminder scheduling (`schedule_gravity_for_user`), callback query handler, Gravity response handlers |
| `schedule_handlers.py` | Schedule setup/management: `enable/disable_reminders`, `setup_schedule_cmd`, `set_schedule_cmd`, `toggle_water_cmd` |
| `admin_handlers.py` | Admin/reporting: `mydata`, `update_cmd`, `weekly_report_cmd`, `timezone_cmd`, `handle_location`, `debug_analytics_cmd` |
| `handlers.py` | Re-export shim — imports from all 6 handler modules (backwards compat for `main.py`) |
| `db_operations.py` | All Supabase queries; no direct psycopg2 |
| `supabase_client.py` | Supabase singleton client |
| `ai.py` | OpenAI plan generation (`gpt-4o-mini`), JSON parsing utilities |
| `gravity_ops.py` | Logs interactions to `progress` table + `user_analytics` |
| `messages.py` | Multi-language strings (`MESSAGES`, `FIELD_LABELS`, `VALUE_LABELS`, `MEAL_LABELS`) |
| `utils.py` | `check_can_regenerate_plan`, `parse_plan`, `he_line`, `generate_plan_with_ai`, etc. |

### SQL / Migrations

All migration files live in `supabase/migrations/`. SQL files are **core logic** and must be reviewed alongside Python files for any DB-related change.

### State Constants (all in `config.py`)

- **Onboarding** (0–14): `TOS_ACCEPT=0`, `LANG_SELECT=1`, `AGE=2`, `HEIGHT=3`, `WEIGHT=4`, `GENDER=5`, `FITNESS_LEVEL=6`, `GOAL=7`, `CONFIRM=8`, `FIX_FIELD=9`, `EXTRA=10`, `CONFIRM_EXTRA=11`, `DIET_STYLE=12`, `WATER_REMINDER_CHOICE=13`, `WORKOUT_SCHEDULE=14`
- **Schedule setup** (16–23): `SCHEDULE_SETUP_MEALS=16` … `SCHEDULE_SELECT_TYPE=21`, `SCHEDULE_SET_TIME=22`, `RESET_CONFIRM=23`
- **Adjust flow** (100–102): `ADJUST_SELECT`, `ADJUST_DETAILS`, `ADJUST_CONFIRM`
- **Replan flow** (200–203): `REPLAN_INPUT=200`, `REPLAN_SELECTION=201`, `REPLAN_INPUT_MEALS=202`, `REPLAN_INPUT_WORKOUTS=203`
- **Update profile**: `UPDATE_FIELD=300`

### Cross-Module Imports

- `onboarding` → `plan_handlers` (generate_and_send_plan, show_or_build_plan), `reminder_handlers` (schedule_gravity_for_user)
- `schedule_handlers` → `reminder_handlers` (schedule_gravity_for_user)
- `admin_handlers` → `reminder_handlers`, `plan_handlers`
- `reminder_handlers` ↔ `plan_handlers` (ai_check_compliance, get_active_plan_text)

### Key Conventions

- Hebrew is the primary/fallback language (`lang="he"`). RTL constants (`RLE`, `PDF`, `RLM`) in `config.py`.
- Handler order in `main.py` matters: `CallbackQueryHandler` before catch-all gravity text handler.
- Plan rate limit: 2 plans per 72h window; admins (in `utils.ADMIN_IDS`) bypass.
- `get_user_profile()` returns both `height_cm`/`weight_kg` AND `height`/`weight` for compat.

---
> Source: [Avivporze/Coachify](https://github.com/Avivporze/Coachify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
