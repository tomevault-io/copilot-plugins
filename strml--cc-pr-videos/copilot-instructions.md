## cc-pr-videos

> A Claude Code plugin that automatically records browser demo videos of pull requests.

# cc-pr-videos

A Claude Code plugin that automatically records browser demo videos of pull requests.

## What it does

When `gh pr create` succeeds and the PR URL is written to a sentinel file, a Stop hook launches a background recorder that:
1. Gathers PR context via `gh pr view`
2. Probes common dev server ports (3000, 3001, 5173, 5174, 4173, 8080, 8000, 4200)
3. Spawns a nested `claude --print` session using the `agent-browser` CLI to record a `.webm` demo
4. Fires a macOS notification when the recording is saved to `.tmp/pr-videos/`

## Project structure

```
commands/init.md          # /pr-videos:init command — sets up .claude/demo.json in a target project
hooks/hooks.json          # Stop hook definition (fires stop-trigger.sh on session end)
scripts/
  stop-trigger.sh         # Reads sentinel /tmp/claude/pr-demo-pending, launches recorder
  recorder-run.sh         # Gathers PR context, runs nested Claude + agent-browser
README.md                 # Plugin install and config docs
```

No build system, no tests. This is a pure shell plugin.

## Key files

| File | Purpose |
|------|---------|
| `hooks/hooks.json` | Declares the Stop hook pointing to `stop-trigger.sh` |
| `scripts/stop-trigger.sh` | Checks sentinel, de-duplicates stop hook loops, launches recorder via `nohup` |
| `scripts/recorder-run.sh` | Main recorder: probes ports, reads `.claude/demo.json`, builds agent-browser prompt, writes feedback log |
| `commands/init.md` | Slash command that configures the plugin in a consumer project |

## Sentinel pattern

The plugin uses a sentinel file at `/tmp/claude/pr-demo-pending` (containing the PR URL) to bridge from the main Claude session (sandbox-restricted) to the Stop hook (runs outside sandbox with access to `~/.claude.json`).

- **Write** the sentinel after `gh pr create` succeeds.
- **Stop hook** reads and deletes it, then launches the recorder via `nohup ... &`.
- The `stop_hook_active` field in hook input prevents infinite re-triggering.

## Consumer project config (`.claude/demo.json`)

```json
{
  "baseUrl": "http://localhost:3000",
  "auth": {
    "loginUrl": "/login",
    "username": "user@example.com",
    "password": "pass"
  },
  "browserState": ".claude/demo-browser-state.json",
  "hints": "The feature is at /settings/billing"
}
```

Omit `auth` entirely if no login is required. `browserState` and `demo-feedback.log` should be in `.gitignore`.

## Prerequisites for end users

- `agent-browser` on PATH
- `gh` CLI authenticated
- `jq` installed
- Running dev server

## Editing guidelines

- Keep scripts POSIX-compatible where possible; macOS-only features (e.g., `osascript`) are acceptable since this plugin targets macOS.
- The recorder runs with `--dangerously-skip-permissions` and `--no-session-persistence` — scope the nested Claude call tightly.
- `set -euo pipefail` is used in `recorder-run.sh`; be careful with commands that can return non-zero without being fatal.
- `stop-trigger.sh` intentionally does NOT use `set -e` — it must be resilient.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/STRML)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/STRML)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
