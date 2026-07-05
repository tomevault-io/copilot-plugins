## magic-compact

> Magic Compact is a lossless context compression plugin for OpenCode. See `README.md` for the user-facing overview, features, installation, source map, and pruning rules.

# Project

Magic Compact is a lossless context compression plugin for OpenCode. See `README.md` for the user-facing overview, features, installation, source map, and pruning rules.

This file is the development reference for agents working on this repo.

---

## Architecture

Magic Compact performs backup-first, per-turn conversation compaction in the current session.

### Runtime Surfaces

- `/magic-compact [N]` slash command.
- `/magic-stats` slash command.
- `read_omitted_content` tool.

### Data Boundaries

- Session data remains in OpenCode message and part tables.
- Large omitted content is stored in filesystem JSON files under OpenCode's data directory.
- Session metadata stores small Magic Compact state such as `compactionCount`.

### Goals

- Preserve the current conversation structure while reducing context size.
- Preserve user messages exactly.
- Replace old assistant turns with local per-turn summaries.
- Keep useful tool calls visible while removing bulky tool I/O.
- Store omitted tool I/O outside context but make it retrievable.
- Keep a backup session available for failure recovery.

---

## Behavior Notes

Non-obvious behavior and design rationale. See `src/` for implementation details.

### Turn Selection

- A turn is one or more adjacent user messages plus all following assistant messages before the next user group.
- Consecutive user/no-reply messages are grouped into the same turn.
- A trailing user-only turn does not count against `N`.
- Only turns with assistant messages are summarized.
- `N` preserves the most recent assistant turns in the current uncompacted range.

### Recompaction and Boundary Detection

Previously summarized turns are preserved as-is. Recompaction only considers unsummarized turns at or after the latest boundary.

Boundary detection:

1. Build chronological turns.
2. Scan turns newest to oldest for a user text part with `metadata.magicCompact.boundary === true`.
3. Use that turn as the start of the uncompacted range.
4. Then ignore a trailing assistantless turn for assistant-turn counting.

Boundary detection must happen before removing a trailing assistantless turn because an all-turn compaction stores the boundary as a no-reply user message.

### Summarization Design

- Summaries are per-turn, not monolithic.
- User messages are preserved exactly.
- Summary generation happens in an ephemeral session so the compaction prompt and assistant stream are not shown in the main session.
- Each summary is written as a text part on the first assistant message in the summarized turn.
- Summary parts use deterministic IDs and `metadata.magicCompact.summary === true`.
- The XML prompt includes only turns to summarize and, when needed, the next user turn to mark the boundary. User text in the prompt excludes synthetic text and is truncated to the first line or first 300 characters, whichever is shorter.

### Omission Cache

- Location: `${XDG_DATA_HOME:-~/.local/share}/opencode/storage/magic-compact/{sessionId}.json`.
- IDs: sequential `omitted-001`, `omitted-002`, ... using cache `nextId`.
- The current session cache is the active cache on success.
- The backup receives a cache copy before mutation so omitted content remains available if the backup is promoted.

### Pruning Decision Rationale

- `read` always omits output because stale file contents are reloadable via a new tool call.
- `todowrite` and `skill` outputs are discarded without caching because they are redundant or reloadable.
- `task` uses a higher output threshold (512 words / 4096 characters) because subagent results are often high-signal.
- `question` input and output are always preserved because they capture explicit user decisions.
- Only completed tool parts are pruned. Error tool calls are preserved as-is. Pending and running tool parts won't be encountered during operation.

### Error Handling Contract

Any LLM failure, invalid XML, SDK failure, cache failure, or pruning failure aborts the compaction attempt.

- If a backup exists, it is promoted to replace the failed original session.
- The command hook throws so OpenCode does not continue sending the slash command to the LLM.

---
> Source: [aerovato/magic-compact](https://github.com/aerovato/magic-compact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
