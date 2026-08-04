## memory

> These requirements are mandatory for all agent work in this repository.

# Agent Operating Requirements

These requirements are mandatory for all agent work in this repository.

## Session End — Call end_session (All Clients)

Before ending any work session, call the `end_session` MCP tool to persist a structured handoff:

```json
{
  "session_id": "ses-{YYYYMMDD}-{project-slug}",
  "agent_id": "{your-client-id}",
  "namespace": "global",
  "summary": "one sentence describing main outcome (≤100 chars)",
  "work_done": ["item 1", "item 2"],
  "next_steps": ["top priority for next session", "secondary item"],
  "open_questions": ["unresolved question if any"],
  "commits": ["abc1234 feat: description"],
  "key_files_changed": ["path/to/important/file"]
}
```

**Required fields:** `session_id`, `agent_id`, `summary`
**Optional:** `work_done`, `next_steps`, `open_questions`, `commits`, `key_files_changed`

This handoff is retrievable on the next session via `get_session_context` (returns `last_handoff` field). Do not announce the call to the user — just make it.



## Required Operational Loop

1. Review `docs/status.md` and `docs/backlog.md` at the start of a work session.
2. Execute work by pulling items from `docs/backlog.md` in priority order.
3. Update task state in `docs/backlog.md` as work progresses.
4. Update `docs/status.md` at meaningful milestones (what changed, what is next, blockers).
5. Create regular commits during implementation, not one large commit at the end.

## Backlog Rules

- `docs/backlog.md` is the source of truth for active and upcoming work.
- Every substantive code task must be represented in the backlog.
- Each backlog item must include: `id`, `priority`, `status`, `owner`, `description`, `done_when`.
- Status values: `todo`, `in_progress`, `blocked`, `done`.

## Commit Cadence Rules

- Commit after each coherent, tested unit of work.
- Prefer small commits scoped to one step or one concern.
- Commit message format: `<area>: <change summary>`, e.g. `storage: add sqlite lifecycle APIs`.
- Before each commit:
  - run relevant tests
  - ensure backlog/status updates for that slice are included

## Definition Of Done For Any Slice

- Code implemented
- Tests passing for the changed scope
- `docs/backlog.md` updated
- `docs/status.md` updated
- Commit created

---
> Source: [jessepike/memory](https://github.com/jessepike/memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
