## claude-code-scheduler

> A Claude Code plugin that adds scheduled tasks. Shell scripts + Python 3 only, no other dependencies. Users invoke `/schedule` and Claude orchestrates the scripts.

# CLAUDE.md

## What this is

A Claude Code plugin that adds scheduled tasks. Shell scripts + Python 3 only, no other dependencies. Users invoke `/schedule` and Claude orchestrates the scripts.

## Architecture

```
skills/schedule/SKILL.md        <- Skill that Claude follows when /schedule is invoked
scripts/task-manager.sh         <- CRUD for task JSON in ~/.claude-scheduler/tasks/
scripts/platform-scheduler.sh   <- launchd (macOS) / crontab (Linux) registration
scripts/run-task.sh             <- Execution wrapper: reads task JSON, runs claude -p, logs, notifies
scripts/notify.sh               <- Cross-platform desktop notifications (osascript / notify-send)
tests/regression.sh             <- Regression tests for known edge cases
```

Task data lives in `~/.claude-scheduler/tasks/<id>.json`. Logs go to `~/.claude-scheduler/logs/<id>/`.

## Known constraints

- `run-task.sh` cannot be called from within an active Claude session. It invokes `claude -p` which fails as a nested process. The SKILL.md instructs Claude to use the register-then-unregister pattern for "run now" requests instead.
- Cron/launchd minimum granularity is 1 minute. Sub-minute scheduling is not possible.

## Key conventions

- All JSON manipulation uses `python3 -c` one-liners (not jq, not sed). This keeps escaping safe.
- Update args are passed as key/value argv pairs, not newline-delimited strings (see `cmd_update` in task-manager.sh).
- Cron day-of-month and day-of-week use OR semantics (standard cron behavior), not AND. See the `day_week_entries` logic in platform-scheduler.sh.
- launchd plists are generated via Python heredoc in `platform-scheduler.sh`. The heredoc uses `$PYTHON - "$arg1" "$arg2" << 'PYEOF'` syntax (note the `-` for stdin).
- Scripts output JSON to stdout for machine consumption and errors to stderr.

## Testing

```bash
# Syntax check all scripts
bash -n scripts/*.sh tests/*.sh

# Run regression tests
tests/regression.sh

# Validate plugin metadata
python3 -m json.tool .claude-plugin/plugin.json
```

The regression suite covers:
- Multiline prompt preservation through update
- `notify=false` suppressing preflight failure notifications
- launchd day-of-month/day-of-week OR semantics (macOS only, skipped on Linux)

CI runs these on both Ubuntu and macOS via `.github/workflows/ci.yml`.

## Common tasks

**Adding a new script flag**: Add the flag to the `while` loop in the relevant `cmd_*` function, then pass it through to the Python block. Follow the existing argv-pair pattern in `cmd_update`.

**Changing the plist template**: Edit the `generate_plist` Python heredoc in `platform-scheduler.sh`. Run `plutil -lint` on the output to validate. The `test_launchd_dom_dow_or_semantics` regression test verifies interval generation.

**Adding a new regression test**: Add a `test_*` function in `tests/regression.sh` and call it from `main()`. Use `new_tmp_home` to get an isolated `$HOME` so tests don't touch real task data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intertwine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/intertwine)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
