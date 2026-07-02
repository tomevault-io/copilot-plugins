## clawphone

> Gitignored personal context files live in `context/`. At the start of each session,

# Project context for ask_claude invocations

## Personal context

Gitignored personal context files live in `context/`. At the start of each session,
read all `.md` files in that directory — they contain Adam's personal preferences,
communication templates, and standing instructions that should inform your responses.

This Claude is invoked headlessly by `main.py` to answer questions the caller asks over the phone (via the `ask_claude` tool exposed to the OpenAI realtime model). Optimize for short, spoken answers — the result is read aloud, so prefer plain prose over markdown tables, bullet lists, or code blocks unless the caller specifically asks.

## MCP servers available

- **google-workspace** — Gmail + Google Calendar (read/write). Use for "what's on my calendar", "did I get an email from X", sending mail, scheduling events, etc.
- **joplin** — Personal notes stored in Joplin, synced via S3. Use for "do I have a note about X", "what's on my todo list", reading or creating notes.

## Joplin: prefer `search_notes` over `read_notebook`

`mcp__joplin__read_notebook` returns at most 100 notes per call **with no pagination** — so for large notebooks it silently truncates. Don't use it to count, list, or filter notes in any notebook that might have more than ~50 entries.

For any "count", "list", "find incomplete todos", or "what's on my X list" query, use `mcp__joplin__search_notes` with [Joplin search syntax](https://joplinapp.org/help/apps/search):

| Goal | Query |
|---|---|
| Incomplete todos in a notebook | `notebook:"== Main ==" iscompleted:0 type:todo` |
| Completed todos in a notebook | `notebook:"== Main ==" iscompleted:1 type:todo` |
| All notes in a notebook (no filter) | `notebook:"NotebookName"` |
| Notes containing a word | `keyword` (full-text search) |
| Notes tagged X | `tag:X` |
| Notes updated since a date | `updated:20260501` |

Combine operators with spaces (implicit AND). Quote notebook names that contain spaces or symbols.

If `search_notes` reports `has_more`, paginate using its `page` parameter — don't report a truncated count as the total.

## Receipt printer

A thermal receipt printer is available at `http://localhost:8000/print`. POST with multipart/form-data, field name `raw_text`, value = the text to print. Example:

```
curl -X POST http://localhost:8000/print -F "raw_text=Hello world"
```

Use this when the caller asks to print something that requires fetching data first — e.g. "print my todo list" (fetch from Joplin, then print) or "print tomorrow's schedule" (fetch from Calendar, then print).

---
> Source: [derwiki/clawphone](https://github.com/derwiki/clawphone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
