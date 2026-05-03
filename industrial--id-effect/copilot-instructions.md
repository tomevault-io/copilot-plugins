## no-direct-events-jsonl-edits

> Never edit vault/Tasks/events.jsonl, .tool-agent-forum/events.jsonl, or tool-llm-git-context session logs by hand — use the tool CLIs only.


# Append-only event logs — no direct file edits

**Do not** mutate these files with editor or patch tools (`apply_patch`, `Write`, `StrReplace`, or equivalent):

- **`.tool-llm-git-context/vault/Tasks/events.jsonl`** (tool-tasks)
- **`.tool-agent-forum/events.jsonl`**
- **`.tool-llm-git-context/vault/Logs/*.jsonl`** (Cursor hook session log — see **`.cursor/rules/llm-git-context.mdc`**; legacy paths under `worktrees/**/Logs/` or `events.jsonl` should not be hand-edited either)

They are **append-only** application state. Hand-editing risks torn lines, invalid JSONL, broken event order, and concurrent-writer corruption.

## Allowed changes

- **Mutations** only through the official binaries:
  - Tasks: `devenv shell -- cargo run -p tool-tasks -- …` (see **`tool-tasks.mdc`**).
  - Agent forum: `devenv shell -- cargo run -p tool-agent-forum -- …`
  - LLM session hook log: Cursor hooks call `tool-llm-git-context append-event`, or run manually with stdin JSON (see **`llm-git-context.mdc`**); never edit the JSONL file in the editor.
- **Read-only** inspection (e.g. `read_file`, terminal `tail`) is fine for debugging; prefer **`--toon`** CLI output when you need structured data.

## If the user asks to “fix” or “clean” a log

Use **`tool-tasks clean`** / documented CLI flows, or ask the user to run backups and use the supported maintenance commands — **do not** rewrite `events.jsonl` directly.

---
> Source: [Industrial/id_effect](https://github.com/Industrial/id_effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
