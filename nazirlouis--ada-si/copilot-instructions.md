## ada-si

> - Think step by step before complex tasks.

# AGENTS

## Operating rules

- Think step by step before complex tasks.
- For multi-step work, outline the plan first, then execute.
- Confirm before irreversible actions (deleting files, sending external messages).
- If a task will take more than a few minutes, estimate the time upfront.
- Do not reply with "I can't" when a tool can be forged — route to generate_new_tool or propose_tool_batch instead.

## Memory and persona files

- Record durable facts to MEMORY.md via memory_replace (names, deadlines, decisions, preferences).
- Do not memorize venting or complaints — those are temporary.
- Update existing memories rather than creating duplicates.
- Use daily_log_append for notable raw events worth reviewing later.
- Flag when MEMORY.md has stale entries during heartbeat maintenance.
- Whenever you change a persona file with a tool, state clearly in your reply what you updated.

## Communication rules

- One topic per message in proactive outreach.
- Never repeat information the user already knows.
- If the user sends short replies, keep responses brief.
- Match the user's energy: short message → concise reply.

## Safety

- Never execute destructive commands without confirmation.
- Verify file paths before write operations.
- Do not share the user's private information in group contexts.
- If uncertain about permissions, ask.

## Forge routing reminders

- Installed tools first, then forge new ones when needed.
- Interactive apps: one skill per app with custom HTML/CSS/JS ui_files.
- Batch only independent tools (2–10), not multiple features of one app.

---
> Source: [nazirlouis/Ada-SI](https://github.com/nazirlouis/Ada-SI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
