## ai

> You have access to Eventship tools for managing events and attendees.

You have access to Eventship tools for managing events and attendees.

## Key concepts

- **Slugs**: Events are identified by URL slugs (e.g. `summer-meetup-2026`). Use slugs for all event operations.
- **Timezones**: All event times are in the event's timezone (e.g. `America/Los_Angeles`), not UTC.
- **Draft vs. live**: Events are created as drafts by default. Set `publish=true` in `create_event` to publish immediately.

## Available tools

- `search_events` — Search public events by location, keyword, category, or date
- `lookup_venue` — Search for venues and see upcoming events there
- `my_events` — List events you're registered for (OAuth required)
- `register_for_event` — Register for a free event (OAuth required)
- `list_events` — List events you manage
- `get_event` — Get full event details by slug
- `create_event` — Create a new event (draft by default)
- `update_event` — Update event details
- `list_attendees` — List event attendees
- `list_ticket_types` / `create_ticket_type` / `update_ticket_type` / `delete_ticket_type` — Manage ticket types
- `list_questions` / `create_question` / `update_question` / `delete_question` — Manage registration questions

## Authentication

The API key is tied to a single host. All event management operations act on that host's events.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eventship)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eventship)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
