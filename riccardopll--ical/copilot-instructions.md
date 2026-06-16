## ical

> Use the ical CLI to inspect local Apple Calendar data. Trigger for listing calendars, checking upcoming events, querying events by date range, filtering by calendar id, or providing calendar-aware local context.


# iCal CLI

Use `ical` to inspect local Apple Calendar data and return concise summaries.

## Workflow

1. Discover calendars before applying a calendar filter:

   ```bash
   ical calendars
   ```

2. Query events with the narrowest useful range:

   ```bash
   ical list
   ical list --days 7
   ical list --from 2026-03-10 --to 2026-03-12
   ical list --from 2026-03-10 --to 2026-03-12 --calendar <id>
   ```

3. Summarize command output for the user instead of dumping long event lists unless they ask for raw output.

## Gotchas

- Date arguments must use `YYYY-MM-DD`.
- Calendar filters use the calendar `id` from `ical calendars`. Calendar names are not unique.
- The default `ical list` window starts today and runs through tomorrow.

---
> Source: [riccardopll/ical](https://github.com/riccardopll/ical) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
