## tai

> - Root: repository root (contains `tai` launcher and `lib/` scripts). `.tai_bus/` is created per project for requests/responses/logs.

# tai assistant playbook

## Project basics
- Root: repository root (contains `tai` launcher and `lib/` scripts). `.tai_bus/` is created per project for requests/responses/logs.
- Entry points: `tai` (CLI), `lib/tai-split.sh` (tmux layout), `lib/tai-agent-loop.sh` (worker), `lib/tai-task.sh` (queue), `lib/tai-color.sh` (log coloring), `lib/tai-utils.sh` (helpers).
- Typical flow: inside tmux, run `tai agent` to open panes; queue work with `tai task "…"`. Agent writes to `.tai_bus/logs/agent.log` and responses in `.tai_bus/responses/`.

## Constraints
- Must run `tai agent` inside tmux; otherwise it errors.
- Default shell from `SHELL` or tmux `default-shell`.
- No network installs/updates unless explicitly allowed.
- Respect existing user changes; do not revert unrelated edits.

## Coding guidelines
- Bash scripts use `#!/usr/bin/env bash` and `set -euo pipefail` when executed directly; keep sourcing-safe guards when needed.
- Prefer tmux-friendly commands; avoid disruptive tmux actions (no resets).
- Keep output concise; add brief comments only for non-obvious logic.
- Use ASCII; avoid introducing Unicode unless already present.

## Common commands
- Queue task: `tai task "do something"`.
- Start the agent/log panes: `tai agent`.
- Open the Codex CLI in a tmux split: `tai tui`.
- Send raw input to the `tai-tui` pane: `tai send "…"`.
- View log: `tail -f .tai_bus/logs/agent.log`.
- Status: `tai status` reads `.tai_bus/status.txt`.
- List requests/responses: `ls .tai_bus/requests` / `ls .tai_bus/responses`.

## Testing / verification
- For script changes, sanity-check syntax: `bash -n lib/<file>.sh`.
- Dry-run tmux layout by running `tai agent` inside tmux and confirm panes stay open and colored logs stream.

## Prompting tips (for model)
- When given tasks, include relevant paths (e.g., `lib/tai-split.sh`) and expected behavior.
- If using this file in a session, load once and keep in context; otherwise changes are stateless.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emmacharp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emmacharp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
