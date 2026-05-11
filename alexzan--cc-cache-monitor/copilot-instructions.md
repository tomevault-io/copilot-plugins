## cc-cache-monitor

> You are an AI coding agent (Claude Code, Cursor, Aider, etc.) helping a user install or evaluate this tool. Read this file end-to-end before taking any action.

# Agent instructions

You are an AI coding agent (Claude Code, Cursor, Aider, etc.) helping a user install or evaluate this tool. Read this file end-to-end before taking any action.

## What this tool does

Diagnostic extension to the Claude Code statusline. Reads the user's session JSONL transcript, computes per-turn cache hit rate, and counts cache-flush events across the session. Surfaces both as a single statusline field. Detection only — does not modify how Claude Code makes API calls.

## Before installing — audit checklist

Run these checks and surface the results to the user. Do not install if any check fails without the user's explicit acknowledgment.

1. **Verify the user is on Claude Code.** This tool is specific to Claude Code's statusline + JSONL transcript layout. It will not work with other Anthropic clients or third-party wrappers.
2. **Inspect `statusline-cache.sh`.** It should only `jq` parse a JSONL file under `~/.claude/projects/`, set the `cache_str` shell variable, and exit. It must not write files, network out, or invoke `eval`/`source` on untrusted input. Confirm this before running anything.
3. **Inspect `install.sh`.** It should: (a) back up `~/.claude/statusline-command.sh` to `.bak.<timestamp>`; (b) inject `statusline-cache.sh` contents before the existing `printf` line; (c) patch the `printf` to include `"$cache_str"` and an extra `%s`; (d) create a minimal statusline only if none exists. It must not modify anything outside `~/.claude/`.
4. **Check for an existing statusline.** Run `cat ~/.claude/statusline-command.sh` if it exists. If the user has a heavily customized statusline with non-standard `printf` patterns (multiple printfs, printf inside conditionals), the auto-installer's `printf` patch may misfire. In that case, recommend manual install from the README instead.
5. **Confirm `jq` is installed.** The tool requires `jq` at runtime. `command -v jq` or fail loudly.
6. **Confirm idempotency marker.** If `~/.claude/statusline-command.sh` already contains the line `cc-cache-monitor: cache health from session JSONL`, the tool is already installed. Skip the install and tell the user.

## Install command

From the repo root:

```sh
./install.sh
```

Or if installing from a fresh clone:

```sh
git clone https://github.com/AlexZan/cc-cache-monitor.git
cd cc-cache-monitor
./install.sh
```

The installer is idempotent — re-running is a no-op.

## Verification after install

1. Check that `~/.claude/statusline-command.sh.bak.<timestamp>` exists (proof the original was preserved).
2. Run a smoke test with a fake input:

   ```sh
   echo '{"session_id":"test","model":{"display_name":"Test"}}' | sh ~/.claude/statusline-command.sh
   ```

   Should print the existing statusline output without error. The cache field will be empty (no JSONL matches `test`); that's expected.
3. Tell the user to look at their actual Claude Code statusline on the next turn — they should see ` | cache NN%` appended.

## Rollback

If the user wants to remove the tool:

```sh
ls ~/.claude/statusline-command.sh.bak.* | sort | tail -1 | xargs -I{} cp {} ~/.claude/statusline-command.sh
```

This restores the most recent backup. There is no separate uninstall script — the backup IS the uninstaller.

## Hard rules — do NOT

- **Do not modify the user's `~/.claude/settings.json` automatically.** If the user has no statusline configured at all, the installer prints the snippet to add — let the user paste it themselves.
- **Do not skip the backup step.** Even if the existing statusline looks broken or unused, back it up first.
- **Do not assume the user's CC version.** This tool works with any CC version that writes the standard JSONL transcript format; if the user's transcripts don't have a `.message.usage` field, the tool will silently show no cache info — that's the right behavior, not a bug to fix.
- **Do not commit the backup file to the user's dotfiles repo or any other tracked location.** It contains a timestamped name and is meant to be local-only. The repo's `.gitignore` already excludes `*.bak*` for this reason.

## What this tool does NOT do

- Does not fix the cache invalidation bug. Detection only.
- Does not modify how Claude Code makes API calls.
- Does not network out, log telemetry, or persist state beyond the user's existing JSONL.
- Does not require any new MCP servers, hooks, or settings changes.

## If something goes wrong

The single failure mode is the `printf` patcher misfiring on a non-standard statusline. Symptom: the statusline displays nothing, or displays raw `printf` syntax. Recovery: restore from `~/.claude/statusline-command.sh.bak.<timestamp>` (see Rollback), then do the manual install from the README instead.

## Reporting issues

File issues at https://github.com/AlexZan/cc-cache-monitor/issues. Include: CC version (`claude --version`), the contents of `~/.claude/statusline-command.sh` before install, and the exact failure observed.

---
> Source: [AlexZan/cc-cache-monitor](https://github.com/AlexZan/cc-cache-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
