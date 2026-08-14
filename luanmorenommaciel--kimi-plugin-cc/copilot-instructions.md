## kimi-plugin-cc

> `kimi-plugin-cc` is a Claude Code plugin that bridges Claude Code and the Kimi CLI, enabling users to:

# kimi-plugin-cc — Project Context

## Overview

`kimi-plugin-cc` is a Claude Code plugin that bridges Claude Code and the Kimi CLI, enabling users to:

1. **Review code** with Kimi (standard or adversarial)
2. **Explore codebases** read-only before making changes
3. **Delegate tasks** to Kimi for execution
4. **Generate plans** before implementation
5. **Manage background jobs** with status/result/cancel

## Architecture

```
Claude Code  ──►  slash command  ──►  broker.mjs  ──►  kimi CLI
                    (.md file)         (Node.js)        (local process)
```

The plugin is a Node.js application with a central broker that:
- Parses CLI arguments
- Dispatches to the appropriate lib module
- Manages session state in `~/.kimi-plugin-cc/`
- Captures diffs via git
- Returns structured JSON to Claude Code

## Key Files

| File | Purpose |
|------|---------|
| `plugins/kimi/scripts/broker.mjs` | Central entry point, argument parsing, dispatch |
| `plugins/kimi/scripts/lib/kimi.mjs` | Wraps `kimi` CLI with retry logic and JSONL capture |
| `plugins/kimi/scripts/lib/state.mjs` | Session metadata persistence |
| `plugins/kimi/scripts/lib/git.mjs` | Diff capture and git operations |
| `plugins/kimi/scripts/lib/workspace.mjs` | Repo root detection, session tracking per repo |
| `plugins/kimi/scripts/lib/job-control.mjs` | Background job spawning and cancellation |
| `plugins/kimi/scripts/lib/render.mjs` | Markdown formatting for review/explore output |

## Agent Security Model

Three agent configurations with different tool access:

| Agent | Tools | Use Case |
|-------|-------|----------|
| `coder.yaml` | Full access (WriteFile, Shell, Agent) | Task execution (`/kimi:crank`) |
| `explore.yaml` | Read-only (no WriteFile, Shell, Agent) | Reviews and exploration |
| `plan-sub.yaml` | No shell, no write | Structured planning |

## Session Model

- Sessions are stored in `~/.kimi-plugin-cc/sessions/<uuid>/`
- Each session has: `meta.json`, `output.jsonl`, `kimi.log`, `pid`
- The latest session per repo is tracked in `.kimi/.session`
- Background jobs are detached processes with PID files

## Configuration

- User-level: `~/.kimi/config.toml`
- Project-level: `.kimi/config.toml`
- Plugin data: `~/.kimi-plugin-cc/` (or `KIMI_PLUGIN_DATA` env var)

### Runtime env contract

| Env var | Default | Effect |
|---------|---------|--------|
| `KIMI_PLUGIN_DATA` | `~/.kimi-plugin-cc/` | Root dir for session state |
| `KIMI_DISPATCH_TIMEOUT_MS` | `1800000` (30m) | Hard wall-clock timeout per crank (`runOnce` foreground). On expiry: SIGTERM, then SIGKILL after 2s. |
| `KIMI_IDLE_TIMEOUT_MS` | `300000` (5m) | Idle-output watchdog (foreground `runOnce` and detached background spawn). Kills a crank that stops emitting output. |

A timeout is **terminal** — never retried (only exit-75 transient errors retry, up to 3x). It resolves with the internal sentinel `TIMEOUT_EXIT_CODE` (124) and surfaces to the supervisor as broker exit code **6** with `status: failed`, `reason: timeout`, `committed: false`.

### Broker exit codes (`dispatch`)

| Code | Meaning |
|------|---------|
| 0 | ok / dispatched |
| 2 | origin-diverged |
| 3 | buggy-evals |
| 4 | review-pause (plan/diff review or api-validation concern) |
| 5 | checkpoint-conflict |
| 6 | timeout (wall-clock or idle-output watchdog killed a hung crank) |

## Testing

- Unit tests: `npm test` (Node.js built-in test runner)
- Smoke tests: `npm run smoke` (end-to-end with temp git repo)
- CI: GitHub Actions on push/PR to main

## Release Checklist

1. Update version in `package.json`, `marketplace.json`, `plugin.json`
2. Add release notes to `CHANGELOG.md`
3. Run `npm test && npm run smoke`
4. Commit, tag, push

---
> Source: [luanmorenommaciel/kimi-plugin-cc](https://github.com/luanmorenommaciel/kimi-plugin-cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
