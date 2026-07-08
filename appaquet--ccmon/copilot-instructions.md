## ccmon

> Claude Code + OpenCode session monitor. All timestamps ISO 8601. Watch outputs are NDJSON (one JSON per line, no separators).

# ccmon

Claude Code + OpenCode session monitor. All timestamps ISO 8601. Watch outputs are NDJSON (one JSON per line, no separators).

## Setup

- `npm install`
- OpenCode plugin (sub-100ms state): `cp resources/opencode-plugin/ccmon.ts ~/.config/opencode/plugins/`
  Auto-discovers on next OpenCode start. Falls back to SQLite polling when absent.

## Commands

| Command | Purpose |
|---------|---------|
| `npm run dump` | All projects as JSON. `--watch` for NDJSON stream, `--project <name>`, `--no-filter`, `--max-age <hours>` |
| `npm run status` | Hook event from stdin → appends to `ccmon-status.jsonl` (Stop, PermissionRequest, etc.) |
| `npm run serve` | HTTP + WebSocket server (auto-port, `--port <N>`) |
| `npm run sub` | Connect to server, stream state as NDJSON |
| `npm test` | Run all tests (vitest) |
| `npm run lint` / `lint:fix` | Biome format + lint |
| `npm run typecheck` | `tsc --noEmit` |

## Integration check

After changes to `src/backends/claude.ts` or `src/backends/opencode.ts`:
```bash
npm run dump --no-filter   # must return ≥ 1 project
npm run dump               # stale filter applied
```

## Architecture

**SessionBackend** interface (`src/backends/types.ts`) — seven methods: scan, resolve, enrich, computeLastUpdated, getSubagents, watchForChanges, projectKey. The standalone `buildProjectState(backend, info)` in `src/backends/build-project-state.ts` assembles full `ProjectState` from these focused methods.

| Backend | Source | Change detection | State source |
|---------|--------|-----------------|--------------|
| `ClaudeBackend` | `~/.claude/projects/` filesystem | `fs.watch` on dirs | `ccmon-status.jsonl` hook log |
| `OpencodeBackend` | `opencode.db` SQLite (read-only) | `fs.watch` on plugin status log + polling fallback | Plugin NDJSON log (`~/.local/state/ccmon/opencode-status.jsonl`), SQLite timestamp inference |

States: `running`, `stopped`, `waiting_for_permission`, `error`, `closed`. Plugin enables all 5 for OpenCode; without it only `running`/`stopped` via timestamp inference.

**Config**: `~/.config/ccmon/config.json` (`CCMON_CONFIG` env override). Schema: `{ maxInactivityHours: 1, backends: [{ type, enabled, ...opts }] }`. Defaults to both backends enabled.

**Environment**: `CLAUDE_PROJECTS_DIR` (default `~/.claude/projects`).

## Key files

- `src/sessions.ts` — barrel re-exports from all session modules
- `src/types.ts` — shared types (`ProjectInfo`, `ProjectState`, `SubagentInfo`, `BackendSource`)
- `src/session-core.ts` — status log reading and state resolution (`SessionState`, `StatusEvent`, `resolveState`, `readStatusLog`)
- `src/session-enrichment.ts` — JSONL tail parsing for model/messages/tokens/tasks (`SessionEnrichment`, `scanEnrichment`, `mergeEnrichment`)
- `src/project-utils.ts` — project scanning and filtering (`scanProjects`, `filterStaleProjects`, `disambiguateProjectNames`)
- `src/status-writer.ts` — hook status file writing (`writeStatusEvent`, `writeNotificationStatus`, `mapHookEventToState`)
- `src/backends/claude.ts` — Claude Code backend (filesystem/JSONL, absorbed SessionStore)
- `src/backends/opencode.ts` — OpenCode backend (SQLite read-only)
- `src/backends/types.ts` — `SessionBackend` interface + `BackendConfigEntry` type
- `src/backends/build-project-state.ts` — Shared `buildProjectState(backend, info)` utility
- `src/backends/collect-states.ts` — Shared `collectBackendStates(backends)` for server/CLI
- `src/backends/index.ts` — `createBackends(config)` factory
- `src/server.ts` — HTTP + WebSocket server
- `src/cli.ts` — CLI entry point, all subcommands
- `src/config.ts` — config loading
- `src/watcher.ts` — `fs.watch` + backoff restart logic
- `public/index.html` — single-page vanilla JS dashboard
- `resources/opencode-plugin/ccmon.ts` — OpenCode plugin (zero-dep Bun TS)

---
> Source: [appaquet/ccmon](https://github.com/appaquet/ccmon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
