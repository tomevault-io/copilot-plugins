## timeedit-mcp

> |


# timeedit-mcp skill

## When to use

Use this skill for requests like:
- "What courses/programs are in this calendar?"
- "Show courses under program X"
- "Find events for course Y"
- "When is the exam / re-exam for course Z?"
- "Filter events by date, room, or text"

## What this server does

Reads a TimeEdit `.ics` URL (configured at startup), caches parsed data for up to 24 hours, and exposes filtered schedule tools. Event times are returned in the timezone configured at startup (typically `Europe/Stockholm`). There is intentionally no "dump all events" tool.

## Tool selection guide

1. Need available programs → `list_programs`
2. Need courses in a program → `list_courses_for_program`
3. Uncertain/partial course name → `search_courses_fuzzy`
4. Need events for one course → `list_events_for_course`
5. Need events for one program → `list_events_for_program`
6. Mixed filters (text/location/course/program/date) → `list_events_filtered`

## Recommended workflow

1. **Resolve the course first**
   - Use `search_courses_fuzzy(query, limit=20)`.
   - If the user query is in English but data is Swedish (or vice versa), retry with translated terms.
   - Check the `score` field: 0.9+ is reliable, 0.5-0.9 is uncertain, below 0.5 is likely noise.
   - Do not trust only the top hit; inspect several candidates.

2. **Prefer course code over name**
   - If fuzzy results include a course code (e.g. `FMNF05`), use that in `list_events_for_course`.
   - Code-based queries are more precise than name-based ones.

3. **Use date bounds when the time period is known**
   - Always pass `start_date` / `end_date` (`YYYY-MM-DD`) when the user mentions a specific period.
   - Increase `limit` when searching for exam or re-exam windows.

4. **Zero results**
   - Could mean bad query, or that the course/program is outside the configured ICS source.
   - If fuzzy search also returns nothing, the course is likely not in scope - inform the user.

5. **Re-exam questions**
   - Filter events where `event_type` or `summary` contains: `omtenta`, `omtentamen`, `reexamination`, `re-exam`, `resit`.
   - If none are explicitly labeled, check for multiple `tentamen` entries for the same course and report the later one as "likely re-exam (inferred)".

## Output format (TOON)

All tools return TOON: `{ dataset, columns, rows }`. Rebuild row objects by zipping `columns` with each row.

Some cell values are compact JSON strings - parse with `json.loads()` when structured access is needed. Affected fields: `course_codes`, `course_names`, `programs`, `groups`.

### Event fields (from `list_events_for_course`, `list_events_for_program`, `list_events_filtered`)

| Field | Notes |
|---|---|
| `start` / `end` | Local time in configured timezone |
| `event_type` | Structured type: e.g. `Föreläsning`, `Övning`, `Tentamen`, `Labb` |
| `course_code` | Primary course code for this event (singular) |
| `course_codes` | All course codes sharing this slot (JSON string array) |
| `groups` | Student group identifiers e.g. `D3.07` (JSON string array, separate from course codes) |
| `course_names` | Course name(s) for this slot (JSON string array) |
| `location` | Room/building |
| `summary` | Raw TimeEdit summary string |
| `programs` | Programs this event belongs to (JSON string array) |

### Other datasets

- `programs` → `name`, `event_count`, `course_count` (+ optional `id`)
- `courses` → `course`, `codes`, `programs`, `event_count`
- `course_matches` → `score`, `course`, `codes`, `programs`, `event_count`

## Common pitfalls

- Passing invalid date formats (must be `YYYY-MM-DD`).
- Using empty query strings.
- Calling `list_events_filtered` without at least one of: `text_query`, `location_query`, `course_query`, `program_query`.
- Assuming fuzzy matches are exact - always check `score`.
- Assuming re-exam entries are explicitly labeled; sometimes only generic `Tentamen` appears.
- Forgetting to `json.loads()` nested cell fields like `course_codes` and `groups`.

---
> Source: [ekvanox/timeedit-mcp](https://github.com/ekvanox/timeedit-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
