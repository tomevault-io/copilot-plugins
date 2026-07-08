## automatic

> SMS-based personal assistant for ADHD management. Everything is a **nag** on a single **today list**: you capture items (with `.. `), each gets an **expire time**, and you're reminded on a **Zeno's-paradox cadence** — per-item nags that accelerate as each item's expire time approaches — then, once an item is overdue, a jittered ping every 30–45 minutes until it's done. Also handles Gmail action-item extraction, scheduled basement-light flashes, and morning briefings. Built with FastAPI, PostgreSQL, Twilio, OpenAI GPT-4o, and Gmail IMAP.

# ADHD SMS Bot

SMS-based personal assistant for ADHD management. Everything is a **nag** on a single **today list**: you capture items (with `.. `), each gets an **expire time**, and you're reminded on a **Zeno's-paradox cadence** — per-item nags that accelerate as each item's expire time approaches — then, once an item is overdue, a jittered ping every 30–45 minutes until it's done. Also handles Gmail action-item extraction, scheduled basement-light flashes, and morning briefings. Built with FastAPI, PostgreSQL, Twilio, OpenAI GPT-4o, and Gmail IMAP.

> Reminders and exercise tracking were removed — the system is nags-only. The `Reminder`/`ExerciseLog` models and their intents no longer exist; their tables linger as legacy.

## Architecture

Four Docker services (`docker-compose.yaml`):
- **api** (port 8000): FastAPI SMS webhook (`/sms`) receives Twilio POSTs
- **scheduler**: Background loop (every `TICK_SECONDS=60`s) fires due items + Gmail sync every 30min
- **ui** (port 8081): Web dashboard for viewing/deleting items
- **db**: PostgreSQL 16

## Database Tables

| Table | Purpose |
|---|---|
| `nag_schedules` | The core model — every today-list item (user-created + Gmail action items) |
| `scheduled_flashes` | One-time basement-light flashes ("flash lights at 9pm") |
| `pending_confirmations` | Stores confirmation/follow-up requests, incl. `set_deadline` and undo (10-min TTL) |
| `processed_emails` | Tracks Gmail Message-IDs to prevent re-processing |
| `app_state` | Key-value scheduler state (e.g., "briefing_last_sent_date", "calendar_last_imported_date" for the daily calendar import, "last_nag_sent_at" for the global nag-rate gate, "user_context" for the latest location/intent. Legacy keys from the retired digest model — "next_digest_at", "next_overdue_ping_at", "pending_burst", "burst_last_sent" — may linger but are unused) |
| `sms_log` | Full audit log of all inbound/outbound SMS |
| `checklists` | Named, one-off checklists (created via `#newlist`) |
| `checklist_items` | Items belonging to a `checklists` row (ordered by `position`) |

Legacy tables still in DB but unused by code: `reminders`, `exercise_log`, `action_items`, `recurring_schedules`, `daily_checklist_items`.

## Key Concepts

### Light flashes (`app/models.py: ScheduledFlash`)
"flash lights at 9pm" → `flash_lights` intent → a `ScheduledFlash` row with `fire_at`. The
scheduler's `fire_due_flashes` triggers `_flash_basement_light()` (IFTTT webhooks) at the
time, marks it `done`, and sends a short confirmation SMS. This is the only surviving piece
of the old reminder/event-pair light-flash behavior.

### Nags (`app/models.py: NagSchedule`)
Nags are the unified model for both user-created nags and Gmail-extracted action items. Each item has an **expire time** (`deadline_at`) and nags itself on a **Zeno's-paradox cadence**: each successive nag waits a random 25–50% of the time remaining until the expire time, so reminders accelerate (with jitter) as the deadline nears, then flatten to a jittered every-30–45-minutes ping once overdue.

**Timing concepts:**
- **Expire time** (`deadline_at`): when the item is "due." For one-shots, defaults to 11 PM; for recurring, computed each cycle as the cron start + `deadline_offset_minutes`.
- **Recurrence** (`cron_expression` + `repeating`): how often cycles repeat (e.g., "weekdays at 9am").

**Nag lifecycle (state machine across `scheduler.py` fire functions):**
1. **Dormant** (`active_since=NULL`): waiting for `next_nag_at` to arrive
2. **Cycle start** (`fire_cycle_starts`): when `next_nag_at` passes, sets `active_since=now`, `nag_count=0`, `snooze_count=0`; for recurring sets `deadline_at=now+deadline_offset_minutes`; then points `next_nag_at=now` so the first Zeno nag fires the same tick. (While active, `next_nag_at` is the per-item Zeno send clock; the next cron cycle is recomputed fresh on check-off / overnight rollover.)
3. **Active**: `fire_due_nags` sends Zeno-spaced nags until the item is checked off or rolled over
4. **Check-off / completion**: ends the item for today (recurring → next cycle; one-shot → `status="deleted"`)

**Zeno cadence** (`scheduler.py: fire_due_nags`, `_compute_deadline_interval`):
- Every tick, `fire_due_nags` selects active items whose `next_nag_at` has arrived, sends one nag per item (a rich GPT-written deadline message via `openai_client.generate_deadline_nag_message`, plain fallback), and advances each `next_nag_at` by its Zeno interval
- **Interval** (`_compute_deadline_interval`): before the expire time, a random 25–50% of the remaining minutes (accelerating, jittered), clamped to the item's `min_interval_minutes` or the global `DEFAULT_MIN_INTERVAL` floor (5 min). At/after the expire time it returns `OVERDUE_PING_GAP` plus a random 0–50% jitter (default **30–45 min**) — the overdue ping; the jitter makes simultaneously-overdue items drift apart so they nag individually instead of locking into one combined every-30-min list SMS (which reads like the retired digest). No deadline → flat min interval
- **Coalescing + global gate**: items due in the same tick are merged into one numbered SMS (`_combined_nag_message`, with a GPT `generate_nag_plan` line); a single item gets `_single_nag_message`. At most one nag SMS per `GLOBAL_NAG_MIN_GAP` minutes (default 5) across all items, tracked in `app_state` under `last_nag_sent_at`
- **Quiet hours**: a due item's `next_nag_at` is deferred to `QUIET_HOURS_END` rather than firing
- An overdue item keeps pinging every 30–45 min until **checked off** (`completed_at` set) or **snoozed** (snooze pushes `deadline_at` and `next_nag_at` forward). Items still open at the day boundary are handled by the morning rollover, not by overnight pinging (quiet hours covers the gap)
- **Holds during quiet hours** without advancing (resumes at `QUIET_HOURS_END`); the morning rollover then carries anything still-open from the prior day

**Overnight rollover + missed recap** (`scheduler.py: _rollover_missed`, inside `fire_morning_briefing`):
- At the morning briefing, items whose expire time was before the start of today and which weren't completed are **carried forward**:
  - **Recurring** items whose cron fires **later today** reset to dormant so a fresh cycle starts at that cron time (normal `now + offset` expire). Items whose cron **already fired earlier today** (e.g. a 6 AM daily, briefing at 7:30) are activated immediately with their expire time **pinned to exactly 11 PM today** and `next_nag_at=now` so the Zeno cadence resumes (`roll_recurring_to_next_cycle`) — so they appear on today's list instead of being skipped
  - **One-shots** have `deadline_at` re-dated to today 11 PM (stay active)
- A single **MISSED recap** SMS naming the carried items is sent alongside the briefing

**Context-aware surfacing** (`app/context_engine.py`):
- The user texts plain location/intent ("heading to Target", "home for the night") → parsed as the `context_update` intent → stored in `app_state` under `user_context`
- `evaluate_context(db)` asks GPT (`openai_client.py: select_relevant_items`) which open today-list items fit the moment (time of day + context + task type) and **pulls them forward** by setting `next_nag_at=now`, so `fire_due_nags` nags them on the next tick. Runs **only when the user sends a `context_update` SMS** (not on a timer)

**Quiet hours** (all outbound nags):
- No nags sent between `QUIET_HOURS_START` (default 0 = midnight) and `QUIET_HOURS_END` (default 6 = 6 AM) local time
- A due item's `next_nag_at` is deferred to `QUIET_HOURS_END` rather than firing during the window

**Completion-anchored nags** (`anchor_to_completion=True`):
- Next cycle starts relative to when user marks DONE, not the cron schedule
- Uses `cycle_months` or `cycle_days` + `_next_nag_cycle()` with `relativedelta`

**Gmail-sourced nags** (`source="gmail"`):
- Created by `gmail_sync.py` with `repeating=False` (nag indefinitely until done — Zeno cadence toward an 11 PM expire time, then 30–45-min overdue pings)
- `source_ref` stores the email reference string for dedup
- `ProcessedEmail` table tracks Gmail Message-ID headers to prevent re-analyzing emails on restart

**Calendar-sourced nags** (`source="calendar"`, `scheduler.py: fire_calendar_import`):
- Once/day past `CALENDAR_IMPORT_TIME` (default 8:00 AM), today's events from the Google Calendar ICS feed (`morning_briefing.py: fetch_calendar_items`) are added to the today list as **one-shot active items**: a timed event's expire time is its **start time**; an all-day event's is **11 PM**
- Deduped per event per day via `source_ref = "cal:<UID>:<YYYY-MM-DD>"` — a daily-recurring calendar event yields one fresh item per day, but re-running the import the same day adds nothing. The import day is tracked in `app_state` under `calendar_last_imported_date`

### Confirmation Flow
Many actions (cancel, acknowledge, deadline follow-up) go through a two-step confirmation:
1. System fuzzy-matches user text to an item (keyword prefilter → GPT fallback)
2. Creates `PendingConfirmation` with 10-min expiry
3. Sends "Do X? Reply YES to confirm."
4. Next inbound SMS: if starts with "y" → execute; else → decline

### Today list (`.. ` capture)
The today list is a view over nags, not a new table — it's the unified surface for "what do I need to do today."
- **Capture**: text `.. <thing>` (dot-dot-space, prefix-handled in `/sms`). Routed through the nag pipeline (`parse_user_sms("nag me to " + remainder)` → `_handle_create_nag`) so GPT parses any inline deadline/recurrence.
- **Deadline follow-up**: if no deadline/cron is found, the new nag defaults to end-of-day (11pm) and a `PendingConfirmation(action_type="set_deadline")` is created. The next reply is parsed by `_apply_deadline_reply` (`intent_router.py`) — a time sets `deadline_at`, "none"/blank keeps the default.
- **Daily items** = recurring nags on a daily cron — they reappear on the list every day; checking one off ends today's cycle (`_next_nag_cycle` → next day).
- **Check-off**: text `<thing> done` → normal `acknowledge` flow (keyword prefilter → GPT fuzzy match → `execute_acknowledge`). Also a per-item check-off checkbox on the front-page UI (`/nag/done/{id}`).
- **Close out yesterday**: text `yesterday <thing> complete` (prefix-handled in `/sms` → `complete_yesterday`). Stops a prior-day item's leftover overdue pings *without* consuming today's cycle: a recurring item is rolled onto its next legitimate cycle (`roll_recurring_to_next_cycle`, shared with the morning rollover) and stamped `completed_at` = end of yesterday (so `is_done_today` stays False and today's daily still nags); a one-shot is marked done. Distinct from a plain `<thing> done`, which stamps `completed_at=now` and would silence today too.
- **UI**: `/` (the front page) is "Today's List" only (active nags due/scheduled today, via `context_engine.today_items`), each row a checkbox that checks the item off.

### Checklists (`app/models.py: CheckList, CheckListItem`)
Named, one-off lists handled by prefix shortcuts in `/sms` that bypass the GPT intent router (like `#help`). (The old `##` daily-checklist feature was removed — daily recurring nags on the today list cover that role; the `daily_checklist_items` table lingers as legacy.)

**Named lists** (`#newlist` / `#updatelist` prefixes, `CheckList` + `CheckListItem`):
- `#newlist <title>` then one item per subsequent line — creates a `CheckList` with `CheckListItem`s. Title is optional; if blank it defaults to `"List <Mon DD HH:MM AM/PM>"`. Items keep insertion order via `position`.
- `#updatelist` then one item per line — appends items to the most-recently-activated list (`activated_at desc`); errors if no list exists or no items given
- Items are one-off (no daily reset); toggled/deleted via the UI

## SMS Inbound Flow (`app/main.py: /sms`)

```
Twilio POST → /sms
  ├─ From KATHRYN_PHONE (+19739787648)? → Auto-create nag, send confirmation
  ├─ From != USER_PHONE? → Reject
  └─ From == USER_PHONE:
       ├─ Prefix shortcuts (bypass GPT): "#help", "#newlist", "#updatelist", ".. " (capture nag), "yesterday " (close out a prior-day item) → handle directly, return
       ├─ PendingConfirmation(set_deadline)? → parse reply → set deadline on the just-created nag, return
       ├─ PendingConfirmation exists? → Handle YES/NO → execute or decline
       └─ No pending confirmation:
            parse_user_sms(Body) via GPT → structured intent + data
            handle_intent(db, parsed) → dispatch to handler → reply SMS
```

## Scheduler Loop (`app/scheduler.py: main()`)

Each tick (60s):
1. `fire_morning_briefing()` — once/day at BRIEFING_TIME; runs `_rollover_missed()` first and sends the single MISSED recap SMS
2. `fire_calendar_import()` — once/day at CALENDAR_IMPORT_TIME (8am), add today's calendar events to the list
3. `fire_due_flashes()` — trigger any scheduled light flashes whose time has passed
4. `fire_cycle_starts()` — activate dormant nags whose `next_nag_at` has arrived (sets `next_nag_at=now` so the first nag fires this tick)
5. `fire_due_nags()` — Zeno cadence: send one (coalesced) nag per global gate window for active items whose `next_nag_at` has arrived, then advance each by its Zeno interval (jittered 30–45 min once overdue)

(`evaluate_context()` is **not** on the tick — it runs only when the user sends a `context_update` SMS, via `_handle_context_update`.)

Every 30min: `run_gmail_sync()` → fetch emails → GPT extract action items → create nag schedules

On startup: sends recovery notification SMS, runs column migrations.

## Intent Handlers (`app/intent_router.py`)

| Intent | Trigger words | Handler |
|---|---|---|
| `create_nag` | "nag me", "remind me to", "I need to", "bug me"; also the `.. ` capture prefix | `_handle_create_nag` |
| `flash_lights` | "flash lights at 9pm", "blink the lights" | `_handle_flash_lights` |
| `context_update` | plain location/intent ("at the office", "heading to Target") | `_handle_context_update` → stores `user_context`, surfaces relevant items |
| `acknowledge` | "done", "finished", "completed", "<thing> done" | `_handle_acknowledge` → undo confirmation |
| `cancel` | "cancel", "delete", "nevermind", "stop" | `_handle_cancel` → undo confirmation |
| `snooze` | "snooze", "later", "not now" | `_handle_snooze` |
| `list` | "list", "show", "status", "pending" | `_handle_list` |
| `briefing` | "briefing", "what's my day" | `_handle_briefing` |
| `help` | "#help" (prefix, bypasses intent router), "commands" | `_handle_help` |

## Key Files

| File | Purpose |
|---|---|
| `app/main.py` | FastAPI SMS webhook, auto-nag phone handler |
| `app/scheduler.py` | Background loop, all `fire_*` functions, Gmail sync trigger |
| `app/intent_router.py` | All intent handlers, confirmation execution, keyword prefilter, time helpers |
| `app/models.py` | SQLAlchemy models (NagSchedule, ScheduledFlash, PendingConfirmation, ProcessedEmail, etc.) |
| `app/openai_client.py` | GPT intent parsing prompt, action item extraction, fuzzy matching |
| `app/gmail_sync.py` | IMAP fetch, email dedup via ProcessedEmail, creates nag schedules from emails |
| `app/context_engine.py` | Today-list helpers + `evaluate_context` (context-aware surfacing), `user_context` get/set |
| `app/ui.py` | Web dashboard (port 8081) — `/` is the today list, `/lists` checklists, `/nags` raw nags |
| `app/config.py` | All env var loading with file-based fallbacks |
| `app/twilio_client.py` | `send_sms()` wrapper around Twilio REST API |
| `app/morning_briefing.py` | Weather + calendar + market briefing generation |
| `app/database.py` | SQLAlchemy engine, session factory, Base |

## Configuration (`app/config.py`)

All config is via environment variables with sensible defaults. Credentials fall back to reading from files in `/home/iray/`.

Key settings: `DATABASE_URL`, `OPENAI_API_KEY`, `TWILIO_*`, `USER_PHONE`, `USER_TIMEZONE`, `TICK_SECONDS`, `GMAIL_*`, `WEATHERAPI_KEY`, `BRIEFING_TIME`, `BASEMENT_LIGHT_ON/OFF`, `QUIET_HOURS_START`, `QUIET_HOURS_END`, `DEFAULT_MIN_INTERVAL` (Zeno cadence floor in minutes, default 5), `GLOBAL_NAG_MIN_GAP` (min minutes between any two nag SMS, default 5), `OVERDUE_PING_GAP` (base gap in minutes between pings once an item is overdue, default 30; each ping adds a random 0–50% jitter), `CALENDAR_IMPORT_TIME` (daily time to import calendar events, default 08:00). (`DEFAULT_MAX_INTERVAL`, `DIGEST_MIN_GAP`/`DIGEST_MAX_GAP`, and `EXERCISE_*_TIME` remain in config but are unused by the Zeno model.)

## Development Notes

- Database migrations are done inline in `scheduler.py:main()` using `ALTER TABLE ... ADD COLUMN IF NOT EXISTS` (PostgreSQL)
- `_keyword_prefilter()` tries fast substring matching before calling GPT for acknowledge/cancel — saves API calls
- `with_for_update(skip_locked=True)` used in scheduler queries to prevent double-firing
- `_random_nag_time()` picks a random 9am-5pm time when user doesn't specify one
- Auto-nag phone (`+19739787648`) allows external systems to create nags at 2-hour intervals by texting
- Every inbound SMS from the user hits OpenAI for intent parsing; no local pre-parsing (except prefix shortcuts: `.. `, `#help`, `#newlist`, `#updatelist`, `kk`)

---
> Source: [IsaacRay/automatic](https://github.com/IsaacRay/automatic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
