## ironbee-cli

> TypeScript CLI tool that enforces browser-based verification for AI coding agents.

# IronBee CLI — Project Guide

## What This Is
TypeScript CLI tool that enforces browser-based verification for AI coding agents.
When an agent edits code, it cannot complete its task until it verifies the changes
work in the browser via the IronBee browser-devtools MCP server.

Core principle: **Code without verification is incomplete work.**

## Commands
```
ironbee install [project-dir] [--client <name>]   Install hooks and guidance files
ironbee uninstall [project-dir]                   Remove hooks, guidance files, and .ironbee/
ironbee update                                    Update IronBee CLI to the latest version
ironbee status  [project-dir]                     Show verdict status for all active sessions
ironbee verify  [session-id]                      Dry-run verdict validation (Stop hook checks)
ironbee hook session-start                        SessionStart hook — records session start
ironbee hook session-end                          SessionEnd hook — records session end with reason
ironbee hook activity-start                       UserPromptSubmit/beforeSubmitPrompt hook — starts activity tracking
ironbee hook require-verification                 PreToolUse hook — blocks browser tools until verification-start
ironbee hook require-verdict                      PreToolUse hook — blocks edits until verdict submitted
ironbee hook clear-verdict                        PostToolUse hook — called by client automatically
ironbee hook track-action                         PostToolUse + PostToolUseFailure — queues every tool_call as a send_event
ironbee hook verify-gate                          Stop hook — called by client automatically
ironbee hook verification-start                   Start verification cycle (called by agent via Bash)
ironbee hook verification-end                     End verification cycle (called by agent via Bash)
ironbee hook submit-verdict                       Submit verdict (called by agent via Bash, session_id in JSON)
ironbee analyze [session-id]                      Analyze session metrics (single or all sessions)
ironbee enable-backend <runtime>  [--client <n>]  Enable backend verification for a runtime (writes opinionated default verifyPatterns); --client narrows md updates to one client
ironbee disable-backend <runtime> [--client <n>]  Disable backend verification (resets verifyPatterns to []; preserves customizations); --client narrows md updates to one client
ironbee enable-verification       [--client <n>]  Turn enforcement on (default state); --client narrows artifact rerender to one client
ironbee disable-verification      [--client <n>]  Monitoring-only mode (no enforcement; sessions/tools still ship to collector); --client narrows artifact rerender to one client
ironbee queue status  [--session <id>]            Queue status per session (counts, recent dead-letter errors)
ironbee queue drain   [--session <id>]            Synchronously drain pending snapshots
ironbee queue dead-letter list  [--limit N] [--session <id>]
ironbee queue dead-letter stats [--session <id>]  Histogram by category
ironbee queue dead-letter retry <job_id> [--session <id>]
ironbee queue dead-letter clear [--session <id>]
ironbee queue purge --snapshots                   (Dangerous) delete all snapshots without processing
ironbee queue purge --sessions older-than=<dur>   Remove old empty queue/ subdirs (e.g. 14d, 2h)
ironbee process-job-file <path>                   Worker entry — process one snapshot (spawned detached)
```

## Project Structure
```
src/
├── commands/
│   ├── install.ts        # client dispatcher — auto-detects or uses --client flag
│   ├── uninstall.ts      # ironbee uninstall — removes hooks, guidance, .ironbee/
│   ├── update.ts         # ironbee update — self-update via npm
│   ├── hook.ts           # ironbee hook [session-start|session-end|activity-start|require-verification|require-verdict|clear-verdict|track-action|verify-gate|verification-start|verification-end|submit-verdict]
│   ├── status.ts         # ironbee status — lists .ironbee/ session verdicts
│   ├── verify.ts         # ironbee verify — dry-run verdict validation
│   ├── analyze.ts        # ironbee analyze — session metrics (time + verification quality)
│   ├── queue.ts          # ironbee queue [status|drain|dead-letter {list,stats,retry,clear}|purge]
│   ├── backend-toggle.ts # shared helper: applyEnableBackend / applyDisableBackend / knownRuntimes (consumed by enable/disable-backend.ts)
│   ├── enable-backend.ts # ironbee enable-backend <runtime> — opt-in toggle for backend runtime cycles (writes default verifyPatterns + flips runtime markers in installed md files)
│   ├── disable-backend.ts # ironbee disable-backend <runtime> — resets verifyPatterns to []; preserves customizations
│   ├── verification-toggle.ts # shared helper: applyVerificationToggle (artifacts-before-config ordering); consumed by enable/disable-verification.ts
│   ├── enable-verification.ts # ironbee enable-verification — flips verification.enable: true + re-renders client artifacts
│   ├── disable-verification.ts # ironbee disable-verification — flips verification.enable: false (monitoring-only mode)
│   └── process-job-file.ts # ironbee process-job-file <path> — detached worker entry
├── analysis/
│   ├── time-analysis.ts           # session time metrics — pure logic, no side effects
│   ├── verification-quality.ts    # verification quality metrics — pure logic, no side effects
│   ├── code-changes.ts            # code change metrics — file edits, hot files, churn detection
│   ├── fix-effectiveness.ts      # fix effectiveness metrics — success rate, re-fail rate, fix-to-verify ratio
│   ├── scoring.ts               # composite scores — efficiency, quality, confidence
│   └── cross-session.ts         # cross-session aggregate analysis — all sessions in project
├── hooks/
│   └── core/
│       ├── verify-gate.ts            # shared verification logic — returns result, no process.exit
│       ├── clear-verdict.ts          # shared cleanup logic — no stdin, pure logic
│       ├── submit-verdict.ts         # verdict submission — validates, writes, records verdict_write
│       ├── verification-lifecycle.ts # verification start/end with unique IDs + OTEL trace IDs
│       ├── activity.ts               # activity tracking — start/end pairs with duration calculation
│       ├── activity-end.ts           # monitoring-mode Stop core — endActivity + flushInBackground (no verification logic)
│       ├── session-state.ts          # centralized state (state.json) — retries, activeVerificationId, activeTraceId, lastVerdictStatus, activeFixId, activeActivityId, phase, active
│       ├── tool-use-stash.ts          # generic PreToolUse → PostToolUse stash; tmp-backed so OS reclaims orphans (used by Write to remember existsSync across the call boundary)
│       ├── file-diff.ts               # diffLineCounts (LCS) + countLines — used by clear-verdict for file_change line stats
│       ├── required-tools.ts          # alwaysRequired + evidencePaths satisfier — pure logic for the multi-cycle gate
│       └── actions.ts                # session action logger — actions.jsonl read/write
├── clients/
│   ├── base.ts           # IClient interface
│   ├── registry.ts       # REGISTERED_CLIENTS + findClient / detectClients
│   ├── claude/
│   │   ├── index.ts      # ClaudeClient: install + hook delegation
│   │   ├── util.ts       # Claude-specific helpers: getClaudeUserEmail (~/.claude.json),
│   │   │                 # extractMcpServerName (mcp__<srv>__<tool> parser),
│   │   │                 # extractClaudeToolInput (per-tool whitelist projection),
│   │   │                 # classifyTool (taxonomy normalization)
│   │   ├── skills/
│   │   │   └── ironbee-verification.md  # Auto-invoked skill — copied to dist/ at build time
│   │   ├── rules/
│   │   │   └── ironbee-verification.md  # Rule file — copied to dist/ at build time
│   │   ├── commands/
│   │   │   ├── ironbee-analyze.md  # /ironbee-analyze slash command
│   │   │   └── ironbee-verify.md  # /ironbee-verify slash command
│   │   ├── fragments/
│   │   │   ├── skill.node.md        # Node-cycle content spliced into skill on enable-backend node
│   │   │   ├── rule.node.md         # Node-cycle content spliced into rule on enable-backend node
│   │   │   └── command-verify.node.md  # Node-cycle content spliced into /ironbee-verify on enable-backend node
│   │   └── hooks/
│   │       ├── session-start.ts       # Claude adapter → session_start + stdout instructions
│   │       ├── session-end.ts         # Claude SessionEnd → session_end + closes activity
│   │       ├── activity-start.ts      # Claude UserPromptSubmit → starts activity tracking
│   │       ├── activity-end.ts        # Claude Stop (monitoring mode) → endActivity + flushInBackground
│   │       ├── require-verification.ts # Claude PreToolUse → blocks browser tools without verification-start
│   │       ├── require-verdict.ts    # Claude PreToolUse → blocks edits until verdict submitted
│   │       ├── clear-verdict.ts      # Claude adapter → file_change tracking (op + line counts) + verdict clear
│   │       ├── track-action.ts       # Claude adapter → tool_call tracking (bdt_ prefix, nested execute)
│   │       ├── track-action-monitor.ts # Claude adapter (monitoring mode) → lean queue submit + activity-start fallback
│   │       └── verify-gate.ts        # Claude adapter → core result → exit code 0/2
│   └── cursor/
│       ├── index.ts      # CursorClient: install + hook delegation
│       ├── util.ts       # Cursor-specific helpers: classifyTool (taxonomy),
│       │                 # extractCursorToolInput (per-tool whitelist projection)
│       ├── skills/
│       │   └── ironbee-verification.md  # Auto-invoked skill — copied to dist/ at build time
│       ├── rules/
│       │   └── ironbee-verification.mdc  # Rule file (MDC format) — copied to dist/ at build time
│       ├── commands/
│       │   ├── ironbee-analyze/SKILL.md  # /ironbee-analyze command (skill format)
│       │   └── ironbee-verify/SKILL.md  # /ironbee-verify command (skill format)
│       ├── fragments/
│       │   ├── skill.node.md        # Node-cycle content spliced into skill on enable-backend node
│       │   ├── rule.node.md         # Node-cycle content spliced into rule on enable-backend node
│       │   └── command-verify.node.md  # Node-cycle content spliced into /ironbee-verify on enable-backend node
│       └── hooks/
│           ├── session-start.ts       # Cursor adapter → additional_context output
│           ├── session-end.ts         # Cursor sessionEnd → session_end + closes activity
│           ├── activity-start.ts      # Cursor beforeSubmitPrompt → starts activity tracking
│           ├── activity-end.ts        # Cursor stop (monitoring mode) → endActivity + flushInBackground
│           ├── require-verification.ts # Cursor preToolUse → blocks browser tools without verification-start
│           ├── require-verdict.ts    # Cursor preToolUse → blocks edits until verdict submitted
│           ├── clear-verdict.ts      # Cursor adapter → fire-and-forget
│           ├── track-action.ts       # Cursor adapter → postToolUse with bdt_ filtering + MCP normalization
│           ├── track-action-monitor.ts # Cursor adapter (monitoring mode) → lean queue submit + activity-start fallback
│           └── verify-gate.ts        # Cursor adapter → followup_message on block
├── lib/
│   ├── config.ts         # Config loader (global + project, glob pattern matching, getActiveCycles, getRequiredToolsConfig, MCP entry builders)
│   ├── logger.ts         # Log level control + session file logging
│   ├── output.ts         # Colored console output helpers (picocolors)
│   ├── runtime-section.ts # Marker-based runtime section toggling (skill/rule/command md files); fragment loader; RUNTIME_TARGETS map
│   ├── version.ts        # npm registry version check + update notification
│   ├── stdin.ts          # Cross-platform stdin reader
│   ├── collector.ts      # Remote event collector client (sends actions to IronBee Platform)
│   └── telemetry.ts      # Anonymous PostHog telemetry (opt-out: IRONBEE_TELEMETRY=false)
└── queue/                # File-backed job queue — spec: docs/job-queue.md
    ├── index.ts          # public barrel (submit, snapshot, processFile, drain, register, spawnDetachedWorker, maybeAutoFlush)
    ├── types.ts          # Error classes + Job/DeadLetter interfaces + size/rotation constants
    ├── paths.ts          # Project dir resolution, sessionId validation, snapshot filename regex
    ├── submit.ts         # submit() — O_APPEND+O_NOFOLLOW, 4KB size check, UUID v4 ids; calls maybeAutoFlush after append
    ├── snapshot.ts       # snapshot() — atomic rename; ENOENT → null
    ├── process-file.ts   # processFile() — streaming read, group-by-type dispatch, per-job fallback
    ├── drain.ts          # drain() — roll live → process all snapshots → cleanup queue/ only
    ├── flush.ts          # flushInBackground (Stop hook), flushSynchronously (SessionEnd), maybeAutoFlush (size-based, post-submit)
    ├── dead-letter.ts    # Dead-letter writer + rotation (10MB/10k lines, retain 3 newest by mtime)
    ├── worker-log.ts     # Worker log + rotation (10MB, retain 5 by mtime)
    ├── registry.ts       # Handler registry + snapshotRegistry() for per-pass stability
    ├── register-handlers.ts # Wires the send_event handler into the registry at startup
    ├── spawn.ts          # spawnDetachedWorker() — detached child_process per §9.1
    └── handlers/
        └── send-event.ts # Forwards tool_call events to the IronBee Collector via lib/collector

dist/      # Build output (gitignored)
```

## Client Architecture
- `IClient` interface: `name`, `detect()`, `install(projectDir, config?)`, `uninstall()`, `resolveProjectDir()`, `runSessionStart()`, `runSessionEnd()`, `runActivityStart()`, `runActivityEnd()`, `runRequireVerification()`, `runRequireVerdict()`, `runClearVerdict()`, `runTrackAction()`, `runTrackActionMonitor()`, `runVerifyGate()`. `install()` accepts an optional `config` arg so toggle commands can apply the new state before persisting; the install command itself omits it and falls back to `loadConfig(projectDir)`.
- `REGISTERED_CLIENTS` in `registry.ts` — add new clients here when implementing
- `install.ts` auto-detects via `detect()`, or uses `--client <name>` / `--client all`
- Hook runners resolve client via `--client <name>` flag or `IRONBEE_CLIENT` env var
- Project dir resolved via client-specific env var (e.g. `CLAUDE_PROJECT_DIR`) or `process.cwd()`

## Core Architecture
- **Core logic is client-agnostic** — no `process.exit()`, no stdin parsing, no client-specific code
- `runVerifyGate()` returns `{action: "allow"|"block", message?}` — client adapter translates to exit codes
- `actions.jsonl` is the single source of truth for session events (file edits, tool calls, verification markers)
- Config loaded from `~/.ironbee/config.json` (global) + `<project>/.ironbee/config.json` (project override)

## What CursorClient Writes
| File | Purpose |
|---|---|
| `.cursor/hooks.json` | Hooks v1: `sessionStart`, `sessionEnd`, `beforeSubmitPrompt`, `preToolUse`, `postToolUse`, `stop` |
| `.cursor/skills/ironbee-verification.md` | Verification skill |
| `.cursor/skills/ironbee-analyze/SKILL.md` | `/ironbee-analyze` command — LLM-powered session analysis |
| `.cursor/skills/ironbee-verify/SKILL.md` | `/ironbee-verify` command — visual and functional verification |
| `.cursor/rules/ironbee-verification.mdc` | MDC rule with `alwaysApply: true` |
| `.cursor/mcp.json` | MCP servers: `browser-devtools` (PLATFORM=browser) + `node-devtools` (PLATFORM=node), both `npx -y browser-devtools-mcp` (merged) |

## Cursor Hooks
| Hook Event | Handler | Purpose |
|---|---|---|
| `sessionStart` | `session-start` | Records session start, outputs instructions via `additional_context` |
| `sessionEnd` | `session-end` | Records session end with reason, closes open activity |
| `beforeSubmitPrompt` | `activity-start` | Starts activity tracking on each agent turn |
| `preToolUse` (MCP:(bdt\|ndt)_.*) | `require-verification` | Blocks devtools (browser or node) tools if no active verification cycle, injects `_metadata` (mcpServer derived via prefix lookup). Recording check is browser-only — `ndt_*` tools bypass it. |
| `preToolUse` (Write\|StrReplace\|Delete) | `require-verdict` | Blocks file edits if browser tools used without verdict; stashes existsSync for Write so postToolUse can decide create vs update |
| `postToolUse` (Write\|StrReplace\|Delete) | `clear-verdict` | Records file_change with operation + line counts, clears verdict (skips non-code files); replaces the retired `afterFileEdit` (Cursor 2.3.x always-empty old_string regression + 2.6.x batch-edit drops) |
| `postToolUse` (all tools) | `track-action` | Submits send_event job per tool; for bdt_* normalizes MCP name and records in actions.jsonl |
| `postToolUseFailure` (all tools) | `track-action` | Same adapter; `error_message` + `failure_type` + `is_interrupt` folded into `ToolCallAction.error`; `duration` (ms) preserved |
| `stop` | `verify-gate` | Returns `followup_message` to force agent back into loop when verification fails, ends activity |

**Blocking mechanism:** Cursor's stop hook cannot return `"permission": "deny"`, but it can return `followup_message` which auto-submits a new prompt to the agent, mechanically preventing completion until verification passes (up to Cursor's `loop_limit`, default 5).

**`afterFileEdit` is intentionally NOT used.** Cursor's official hook for file edits has two known reliability bugs that violate IronBee's verify-gate invariants:
- Cursor 2.3.0–2.3.10: `old_string` was always empty regardless of edit, breaking diff calculation (regression fixed post-2.3.10).
- Cursor 2.6.x (open as of 2026-04): in batch edits within a single agent turn, the hook fires for the first file only; remaining files produce no event. Mid-session 3–4 hour silence windows also reported.

We instead drive file_change recording from `postToolUse` with a matcher (`Write|StrReplace|Delete`), which fires reliably per tool invocation.

## Cursor vs Claude Differences
| Aspect | Claude | Cursor |
|---|---|---|
| Session ID field | `session_id` | `conversation_id` |
| Hook output format | Exit codes (0/2) | JSON stdout (`additional_context`, `followup_message`) or exit codes |
| Stop hook blocking | Mechanical (exit 2 blocks) | Mechanical (`followup_message` forces agent back into loop) |
| MCP config location | `.mcp.json` (project root) | `.cursor/mcp.json` |
| Rules format | `.md` | `.mdc` with YAML frontmatter |
| Env var for project dir | `CLAUDE_PROJECT_DIR` | `CURSOR_PROJECT_DIR` |

## Adding a New Client
1. Create `src/clients/<name>/index.ts` implementing `IClient`
   - `hooks/session-start.ts` → parse stdin → record session_start → output session ID
   - `hooks/session-end.ts` → parse stdin → endActivity → record session_end with reason and duration
   - `hooks/activity-start.ts` → parse stdin → call startActivity (user_prompt source)
   - `hooks/require-verification.ts` → check active verification → block browser tools if none → startActivity fallback on allow
   - `hooks/require-verdict.ts` → check actions.jsonl → block edits if verdict missing after tool use → startActivity fallback on allow
   - `hooks/clear-verdict.ts` → parse stdin → derive operation + line counts → record file_change → call core
   - `hooks/track-action.ts` → parse stdin → filter by bdt_ prefix → record tool_call
   - `hooks/verify-gate.ts` → parse stdin → endActivity → call core → translate result to client format
   - `install()` → write client hook config with `--client <name>` flag, MCP config with `TOOL_NAME_PREFIX=bdt_`
2. Add `new <Name>Client()` to `REGISTERED_CLIENTS` in `registry.ts`
3. That's it — `install`, `status`, `verify` all work automatically

## How install Works
- Auto-detects client(s) via `detect()` — falls back to first registered client
- Each client writes its own hook config and guidance files
- Creates `.ironbee/` runtime directory (shared across all clients)
- Appends `.ironbee/sessions/` to project `.gitignore`

## What ClaudeClient Writes
| File | Purpose |
|---|---|
| `.claude/settings.json` | Hooks (SessionStart, SessionEnd, UserPromptSubmit, PreToolUse, PostToolUse, Stop) + Permissions. Permission set is mode-aware: enabled mode → `mcp__browser-devtools__*`, `mcp__node-devtools__*`, `Bash(ironbee *)`; disabled mode → only the narrow `Bash(ironbee analyze)` (MCP perms stripped, full Bash narrowed so agent gets a permission prompt before invoking other ironbee commands). All entries are merged with existing user settings. |
| `.claude/skills/ironbee-verification.md` | Auto-invoked verification skill |
| `.claude/rules/ironbee-verification.md` | Verification rules + BANNED section |
| `.claude/commands/ironbee-analyze.md` | `/ironbee-analyze` slash command — LLM-powered session analysis |
| `.claude/commands/ironbee-verify.md` | `/ironbee-verify` slash command — visual and functional verification |
| `.mcp.json` | MCP servers: `browser-devtools` (PLATFORM=browser) + `node-devtools` (PLATFORM=node), both `npx -y browser-devtools-mcp` (merged) |

## Claude Code Hooks
| Hook Event | Matcher | Handler | Purpose |
|---|---|---|---|
| `SessionStart` | `""` | `session-start` | Records session start, outputs session ID and instructions |
| `SessionEnd` | `""` | `session-end` | Records session end with reason, closes open activity |
| `UserPromptSubmit` | `""` | `activity-start` | Starts activity tracking on each agent turn |
| `PreToolUse` | `mcp__browser-devtools__.*\|mcp__node-devtools__.*` | `require-verification` | Blocks devtools (browser or node) tools if no active verification cycle (exit 2), injects `_metadata` into tool input. Recording check is browser-only — `ndt_*` tools bypass it. |
| `PreToolUse` | `Write\|Edit` | `require-verdict` | Blocks file edits if browser tools used without verdict (exit 2); stashes existsSync for Write so PostToolUse can decide create vs update |
| `PostToolUse` | `Write\|Edit` | `clear-verdict` | Records file_change with operation + line counts, clears verdict (skips non-code files) |
| `PostToolUse` | `""` (all tools) | `track-action` | Submits send_event job per tool; for bdt_* also records in actions.jsonl + recording state + nested callTool extraction |
| `PostToolUseFailure` | `""` (all tools) | `track-action` | Same adapter; `error` field on stdin populates `ToolCallAction.error`, `tool_response` omitted, nested extraction skipped |
| `Stop` | `""` | `verify-gate` | Validates verification, blocks with exit(2) or allows with exit(0), ends activity |

## MCP Servers

Two server entries are installed, both invoking the same `browser-devtools-mcp` package with different env (`PLATFORM=browser|node`, prefix `bdt_|ndt_`). Either or both may be used per session — multi-cycle gate runs each active cycle in parallel.

### `browser-devtools` (default, always installed)
- Package: `browser-devtools-mcp`
- Server key: `browser-devtools`
- Tool name prefix: `bdt_` (set via `TOOL_NAME_PREFIX` env var)
- Tool format (Claude): `mcp__browser-devtools__bdt_<tool-name>`
- Tool format (Cursor postToolUse): `MCP:bdt_<tool-name>`
- Required tools checked by verify-gate: `bdt_navigation_go-to`, `bdt_content_take-screenshot`, `bdt_a11y_take-aria-snapshot`, `bdt_o11y_get-console-messages`
- Nested tools in `bdt_execute` tool: extracted from `callTool('bdt_tool-name', ...)` in tool_input
- `_metadata` injection: PreToolUse (require-verification) injects `{ projectName, sessionId, activityId, verificationId, traceId, traceState, mcpServer, userEmail?, toolUseId?, collectorUrl?, collectorApiKey? }` into tool input. `traceState` format: `ironbee=prj:<name>;sid:<sid>;aid:<aid>;vid:<vid>` (aid omitted if no active activity) — reflects project > session > activity > verification hierarchy. `mcpServer` is the routed MCP server key (`"browser-devtools"` for `bdt_*` tools, `"node-devtools"` for `ndt_*`). Claude derives it from the tool_name via `extractMcpServerName(mcp__<server>__<tool>)`; Cursor uses prefix lookup (`MCP:bdt_` → browser-devtools, `MCP:ndt_` → node-devtools). `userEmail` is read from state.json (populated at session-start) and omitted when unset. `toolUseId` is the hook-provided `tool_use_id` passed through verbatim so the devtools MCP server-side events can be joined with our `tool_call` events by the collector.
- Env vars: `TOOL_NAME_PREFIX=bdt_`, `TOOL_INPUT_METADATA_ENABLE=true`, `BROWSER_DEVTOOLS_INSTALL_CHROMIUM=true`

### `node-devtools` (always installed; enforcement opt-in)
- Same `browser-devtools-mcp` package, `PLATFORM=node` mode.
- Server key: `node-devtools`. Tool prefix: `ndt_`.
- Tool format (Claude): `mcp__node-devtools__ndt_<tool-name>`. Cursor: `MCP:ndt_<tool-name>`.
- 14 debug tools (V8 inspector wrapper) — Connection (`ndt_debug_connect`/`disconnect`/`status`), Probes (`put-tracepoint`/`put-logpoint`/`put-exceptionpoint`), Probe management (`remove-probe`/`list-probes`/`clear-probes`), Evidence (`get-probe-snapshots`/`get-logs`), Probe-snapshot management (`clear-probe-snapshots`), Helpers (`resolve-source-location`/`add-watch`).
- Required tools (node cycle): `ndt_debug_connect` always; one of probe path (`(put-tracepoint | put-logpoint | put-exceptionpoint) AND get-probe-snapshots`) OR log path (`get-logs`).
- Verdict carries `backend_node_processes_connected`, `backend_node_probes_set: NodeProbeRef[]`, `backend_node_probe_snapshots_collected`, `backend_node_log_errors`. `pages_tested`/`console_errors`/`network_failures` remain browser-only.
- **Enforcement is opt-in:** the MCP server is installed but `backend.node.verifyPatterns` defaults to `[]` (inert). Operator activates with `ironbee enable-backend node` (writes opinionated default patterns; preserves customizations on disable).
- Env vars: `PLATFORM=node`, `TOOL_NAME_PREFIX=ndt_`, `TOOL_INPUT_METADATA_ENABLE=true`.

## Runtime Fragments (skill/rule/command md toggling)
Agent-facing markdown files (skill, rule, `/ironbee-verify` command) carry per-runtime marker blocks that are toggled in place by `enable-backend` / `disable-backend` (and by `install` to sync first-time content with current config state). Lets the source md stay runtime-agnostic; runtime-specific guidance is spliced in only when the cycle is enabled.

Marker syntax (two forms — both supported):

```
<!--IRONBEE:RUNTIME:node-->          (keyless block)
...
<!--/IRONBEE:RUNTIME:node-->

<!--IRONBEE:RUNTIME:node:tldr-->     (keyed block — same runtime can have multiple distinct sections in one file)
...
<!--/IRONBEE:RUNTIME:node:tldr-->
```

States:
- **Disabled** — block contents replaced with a per-runtime placeholder (`PLACEHOLDERS` in `runtime-section.ts`) that warns the agent against invoking `<runtime>_*` tools when the backend isn't actually that runtime.
- **Enabled** — block contents replaced with the matching fragment from `src/clients/<client>/fragments/<base>.<runtime>[.<key>].md`. Fragment files are copied to `dist/` at build time and read via `__dirname` at runtime. Missing fragment for an existing key falls back to the placeholder + a warn log.

Files that carry markers (`RUNTIME_TARGETS` in `runtime-section.ts`):
- Claude: `.claude/skills/ironbee-verification.md`, `.claude/rules/ironbee-verification.md`, `.claude/commands/ironbee-verify.md`
- Cursor: `.cursor/skills/ironbee-verification.md`, `.cursor/rules/ironbee-verification.mdc`, `.cursor/commands/ironbee-verify/SKILL.md`

Fragment files in source must use **`.md`** extension regardless of the target file's extension (Cursor's rule.mdc still pulls from `rule.node.md`); the build's `find … -name '*.md' -o -name '*.mdc'` copies both, but only `.md` fragments are read by the loader. Markers in MDC files render as plain HTML comments, same as in markdown, so the toggle works identically.

Reversibility: markers themselves are preserved on every flip, so `disable-backend` followed by `enable-backend` restores the fragment without re-installing.

## Verification Mode Toggle (enabled vs monitoring-only)
**Canonical spec: [`docs/verification-toggle.md`](./docs/verification-toggle.md)**. Summary below.

Default: enabled. Opt-out via `verification.enable: false` in config or via `ironbee disable-verification`. Toggling re-renders client artifacts (hooks, skill, rule, MCP servers, permissions); the change takes effect on the next agent session (host caches hook config at session start).

**Why install-time gating, not runtime**: each ironbee hook fire spawns a fresh `node` process (cold start ~200–300ms). PreToolUse fires per devtools call and per edit; Stop fires every turn. A runtime config check would still pay the cold-start cost. Install-time gating means hooks that have nothing to do in disabled mode are not registered → host never spawns the process.

**Hook map per mode** (mutually exclusive variants for the same event marked **bold**):

| Phase | Verification ENABLED | Verification DISABLED |
|---|---|---|
| SessionStart / sessionStart | session-start | session-start |
| SessionEnd / sessionEnd | session-end | session-end |
| UserPromptSubmit / beforeSubmitPrompt | activity-start | activity-start |
| PreToolUse devtools | require-verification | — |
| PreToolUse edit | require-verdict | — |
| PostToolUse edit | clear-verdict | — |
| PostToolUse + Failure (all) | **track-action** | **track-action-monitor** |
| Stop / stop | **verify-gate** | **activity-end** |

**Mode-specific commands**:
- `track-action-monitor` — lean PostToolUse: queue submit for non-devtools tools + activity-start fallback (no actions.jsonl writes for devtools, no recording state, no nested `bdt_execute` extract).
- `activity-end` — lean Stop: `endActivity` + `flushInBackground`. Pairs with `activity-start` (semantic naming, mirrors the start/end pattern).

**Artifacts NOT installed in monitoring mode**: `ironbee-verification` skill, `ironbee-verification` rule, `/ironbee-verify` slash command, `browser-devtools` + `node-devtools` MCP servers, MCP permission entries (`mcp__browser-devtools__*`, `mcp__node-devtools__*`). `/ironbee-analyze` slash command stays in both modes. **Bash permission is narrowed**: enabled mode writes `Bash(ironbee *)` (skill expects free invocation of `ironbee submit-verdict` etc.); disabled mode writes only `Bash(ironbee analyze)` so any other `ironbee` invocation triggers a permission prompt — defensive layer that catches agents trying to call `submit-verdict` from cached muscle memory before the §"agent-Bash CLI commands" no-op gate kicks in.

**Agent-Bash CLI commands in disabled mode** — `submit-verdict`, `verification-start`, `verification-end` silently no-op in disabled mode (read `getVerificationEnabled(loadConfig(...))` at the top of the action handler, log a debug line, exit 0 without reading stdin or touching state.json/actions.jsonl/verdict.json). Earlier the design said they "execute normally as orphan events but harmless" — turned out **not** harmless: a fail verdict writes `verdict_write` + `fix_start` + sets `phase: "fixing"`, but disabled mode never closes the fix → dangling state until next session-start reconcile. See `docs/verification-toggle.md` §4.6.

**Toggle ordering** — artifacts before config. If `applyClientArtifacts` throws partway, the on-disk config stays in its old state, so re-running the command (or `ironbee install`) re-converges deterministically. Writing config first would risk a divergent state where config says "disabled" but enforcement hooks remain wired in.

**Detection of monitoring sessions** in `/ironbee-analyze`: zero `verification_requested` events. (Not zero `verification_start` — an enabled-mode session where the agent never edited code also has zero of those, but Stop hook still emits `verification_requested(allow)`.)

## Verification Flow (verify-gate, multi-cycle)
**Canonical spec: [`docs/flow.md`](./docs/flow.md)** — end-to-end runtime flow: hook fire order, per-hook allow/deny branches, multi-cycle gate decision tree, cycle lifecycle, state machine, cleanup paths. The summary below is a quick reference; when in doubt, the spec wins.

A single Stop hook run can drive multiple cycles in parallel — each one of (`browser`, `backend.node`, future runtimes). Activation is per-cycle by file_change pattern match; enforcement is per-cycle; pass/fail combines with AND.

1. Compute active cycles → for each `file_change` since last `verification_requested`, check which cycle's pattern set matches.
2. No active cycle → allow with `no_cycle_active`. (Same intent as today's "no verifyPatterns match" allow.)
3. Per active cycle: tool requirement check via `satisfyRequiredTools(used, cfg)`:
   - Browser → `tool_type="mcp"` + `mcp_server="browser-devtools"`, compare against `browser.alwaysRequired` + `evidencePaths`.
   - Node → `mcp_server="node-devtools"`, compare against `backend.node.alwaysRequired` + probe/log paths.
4. Any cycle fails tools → combined block message listing each cycle's gap (`VERIFICATION REQUIRED` when nothing was used at all; `INCOMPLETE VERIFICATION` for partial).
5. Verdict file exists? Valid JSON?
6. Per active cycle: evidence validity (browser → pages_tested+checks+counts; node → backend_node_processes_connected + probe-or-log evidence). Any failure → block.
7. Status interpretation with **pass override**: agent's `status: "pass"` is honored only if every active cycle's pass criteria hold. If status==pass but evidence fails → override to fail (treated as `verdict_fail`, retry counter increments).
8. After `maxRetries` (single global counter, default 3) → allow but require issue reporting.

## Recording Enforcement
**Browser-only.** Backend cycles never set or check `recordingRequired` / `recordingActive`. Multi-mode sessions (browser + node) follow browser-cycle recording flow; node tool calls don't interact.

When `recording.enable: true` in config:
1. `verification-start` → sets `recordingRequired=true`, `recordingActive=false` (mode-agnostic — set unconditionally; enforcement happens only on `bdt_*` tool calls).
2. `require-verification` (PreToolUse, browser-devtools matcher only) → if `recordingRequired && !recordingActive` → blocks all browser tools except `bdt_content_start-recording`. **Never blocks `ndt_*` tools** based on recording state.
3. `track-action` (PostToolUse) → only `bdt_content_start-recording` / `bdt_content_stop-recording` toggle `recordingActive`. ndt_* tool calls don't touch the flag.
4. `submit-verdict` → if `recordingRequired && recordingActive` → rejects ("stop recording first"). Browser-only concern; node-only sessions never have `recordingActive=true`.
5. After successful verdict → resets `recordingRequired=false`.
6. `reconcileSessionState` → resets both flags to false.

## Activity Tracking
**Canonical spec: [`docs/flow.md`](./docs/flow.md) §6.1.** Summary below.

Measures agent active time per turn via `activity_start` / `activity_end` pairs in `actions.jsonl`. Each activity gets a unique `activity_id` that propagates to child `verification_start/end`, `fix_start/end`, and `file_change` events — establishing the `session > activity > verification+fix` hierarchy.

1. **Primary start**: `UserPromptSubmit` (Claude) / `beforeSubmitPrompt` (Cursor) → `activity-start` hook → records `activity_start` with `source: "user_prompt"` and a new UUID `activity_id`
2. **Fallback start**: `PreToolUse` allow path (require-verification, require-verdict) → records `activity_start` with `source: "pre_tool_use"` if not already active
3. **End**: `Stop` hook (verify-gate) → records `activity_end` with matching `activity_id` **only on allow** — block keeps activity open so retry thinking time is included
4. **State**: `active: boolean` + `activeActivityId: string | null` in `state.json` — prevents duplicate starts (idempotent), enables child events to read the parent activity
5. **Reconcile (session resume)**: `reconcileSessionState` closes abandoned activities (no duration, `reason: "session_reconcile"`) on `SessionStart` with `source` ∈ {startup, resume, clear}
6. **Reconcile (compact)**: `reconcileForCompact` closes activity / verification / fix mid-session with `reason: "compact"` — Stop never fires before context is rewritten, so this prevents dangling cycles. Session-scoped state (retries, lastVerdictStatus, verdict file) is preserved.
7. **Block paths**: `startActivity` is only called on allow paths — blocked tool calls do not start activity

## Configuration (`config.json`)
Loaded from `~/.ironbee/config.json` (global) and `<project>/.ironbee/config.json` (project override).
Project config deep-merges over global config (nested `browser` / `backend.<runtime>` blocks merge by key; primitives still shallow-override).

> **Schema note:** The pre-split flat `verifyPatterns` / `additionalVerifyPatterns` at the top level is no longer supported. The config loader fails loudly on legacy shape. Patterns moved under cycle-scoped blocks (`browser.*` / `backend.<runtime>.*`).

```json
{
  "ignoredVerifyPatterns": ["*.test.ts", "*.spec.ts", "docs/**"],
  "maxRetries": 5,

  "browser": {
    "verifyPatterns": ["*.ts", "*.css"],
    "additionalVerifyPatterns": ["src/components/**/*.tsx"]
  },

  "backend": {
    "node": {
      "verifyPatterns": ["server/**/*.ts", "pages/api/**/*.ts"]
    }
  }
}
```

| Key | Description | Default |
|---|---|---|
| `collector.enable` | Master switch. Implicitly `true` whenever the `collector` section exists; set to `false` to suspend the collector while keeping `url` / `apiKey` / `batchSize` around. Absence of the section also disables. | implicit `true` (when section + `url` present) / `false` (otherwise) |
| `collector.url` | Remote collector endpoint URL (IronBee Platform). Required for the collector to be active. | `undefined` (disabled) |
| `collector.apiKey` | API key for collector authentication (`X-API-Key` header) | `undefined` |
| `collector.batchSize` | Max events per POST from `send_event` handler (queue chunks bigger snapshots into N-sized POSTs) | `100` |
| `browser.verifyPatterns` | Glob patterns for files requiring **browser** verification (replaces defaults) | 40+ code extensions (legacy list preserved) |
| `browser.additionalVerifyPatterns` | Extra patterns appended to `browser.verifyPatterns` | `[]` |
| `browser.alwaysRequired` | Browser-cycle required tools (all-of) | `bdt_navigation_go-to`, `bdt_content_take-screenshot`, `bdt_a11y_take-aria-snapshot`, `bdt_o11y_get-console-messages` |
| `browser.evidencePaths` | Alternative tool-satisfaction paths (any-of). Browser uses pure all-of so default is `[]`. | `[]` |
| `backend.<runtime>.verifyPatterns` | Glob patterns for files requiring **backend** verification, per runtime (`node` today). Empty by default — opt in via `ironbee enable-backend <runtime>`. | `[]` |
| `backend.<runtime>.additionalVerifyPatterns` | Extra backend patterns | `[]` |
| `backend.<runtime>.alwaysRequired` | Backend-cycle required tools (all-of) | `node`: `["ndt_debug_connect"]` |
| `backend.<runtime>.evidencePaths` | Alternative tool paths — at least one must be fully satisfied. Each path: `{ name, allOf: (string \| { anyOf: string[] })[] }` | `node`: probe path + log path |
| `ignoredVerifyPatterns` | Patterns to exclude from verification, applied to every cycle (browser + all backend runtimes) | `[]` |
| `maxRetries` | Max verification retry attempts before allowing completion. Single global counter regardless of how many cycles ran. | `3` |
| `verification.enable` | Master switch for enforcement. **Inverse semantics from `recording`/`jobQueue`/`collector`** — verification is the core feature, opt-out by setting `enable: false`. Section absent → enabled. When disabled, ironbee runs in monitoring-only mode: only session/activity/tool_call hooks installed, no skill/rule, no MCP servers, no enforcement. See `docs/verification-toggle.md`. | `true` |
| `recording.enable` | Master switch. Browser-only — backend cycles never trigger recording. Presence opts in; `enable: false` opts out. | implicit `true` (when section present) / `false` (when absent) |
| `jobQueue.enable` | Master switch. Implicitly `true` whenever the `jobQueue` section exists; set explicitly to `false` to disable. Absence of the whole `jobQueue` section also disables. | implicit `true` (when section present) / `false` (when absent) |
| `jobQueue.autoFlushSizeBytes` | Size threshold (bytes) above which `submit()` spawns a detached worker mid-session. `0` / negative disables that check. | `32768` (32 KB) |
| `jobQueue.autoFlushIntervalSeconds` | Interval threshold (seconds since the live `jobs.jsonl` was created) above which `submit()` spawns a detached worker mid-session. Complements size — whichever crosses first triggers flush. `0` / negative disables that check. | `60` |
| `browserDevTools.{mcp,env}` | Override for the browser-devtools MCP server entry. | unset (use defaults) |
| `nodeDevTools.{mcp,env}` | Override for the node-devtools MCP server entry. | unset (use defaults) |

## Event Schema (`actions.ts`)
TypeScript event types mirror the Java `ai.ironbee.domain.event` domain model so `actions.jsonl` entries are directly consumable by the remote collector without transformation.

**Base `ActionEntry`** — fields every event carries (matches Java `BaseEvent`):
- `id: string` (UUID, auto-generated)
- `type: EventTypeValue` (discriminated union, same string values as Java `EventType` enum)
- `timestamp: number` (epoch ms — **not** ISO string)
- `session_id: string`
- `project_name: string`

**Marker interfaces** (mirrors Java marker hierarchy):
- `ActivityAwareActionEntry extends ActionEntry { activity_id }`
- `VerificationAwareActionEntry extends ActivityAwareActionEntry { verification_id }`
- `FixAwareActionEntry extends ActivityAwareActionEntry { fix_id }`

**`baseFields(actionsFile)` helper** — derives `id` + `session_id` + `project_name` from the actionsFile path. Writers spread it into every event construction so the strict `ActionEntry` contract is satisfied without manual plumbing. `appendAction` also enriches missing base fields as a safety net.

**`EventType` const** (`src/hooks/core/actions.ts`) — enum-like object with 12 values (SESSION_START, ACTIVITY_START, ... FILE_CHANGE, TOOL_CALL). JSON values match Java `EventType.getValue()` exactly.

**`Verdict` interface** — typed payload inside `VerdictWriteAction.verdict`. Multi-cycle: `status` and `checks` are shared across cycles; cycle-specific fields are optional and populated only when that cycle is active. Browser fields: `pages_tested?`, `console_errors?` (number), `network_failures?` (number). Node fields: `backend_node_processes_connected?`, `backend_node_probes_set?: NodeProbeRef[]`, `backend_node_probe_snapshots_collected?`, `backend_node_log_errors?`. `NodeProbeRef = { type: "tracepoint" | "logpoint" | "exceptionpoint", location, triggered }`. Java side gains the same fields in a follow-up (D7 in design doc).

**`VerificationRequestedAction.action`** — TS union `"allow" | "block"`, wire-equivalent to Java `VerificationAction` enum. Now also carries `modes: string[]` (registered cycle names — `"browser"`, `"node"`, future runtimes) listing which cycles were active for that gate run.

**`FileChangeAction`** — wire-equivalent to Java `FileChange` event. Carries:
- `tool_name`: raw tool that drove the change (`"Write"` / `"Edit"` for Claude; `"Write"` / `"StrReplace"` / `"Delete"` for Cursor)
- `operation`: `"create" | "update" | "delete"` (`FileChangeOperation` union, mirrors Java enum)
- `lines_added`: number of lines added; `null` when unknown
- `lines_removed`: number of lines removed; `null` when unknown

Line counts are computed via the `diff` package's `diffLines()` (LCS-based) so a single-line replacement inside a block reports `1+1`, not `N+N`. `lines_removed` is `null` for `Write` overwrites (we don't pre-read the prior content) and for Cursor `Delete` (we don't pre-read either, per design — the file vanishes before the postToolUse adapter runs and reading it just to count lines was deemed not worth the I/O). For new-file `Write` and pure additions/removals via diff, both fields carry concrete numbers.

**Tool taxonomy (`src/clients/claude/util.ts` + `src/clients/cursor/util.ts`)** — each adapter has its own `classifyTool()` so the wire conventions (`mcp__<server>__<tool>` for Claude vs. `MCP:<tool>` for Cursor; Claude-only `Skill` bucket; etc.) stay in the client they belong to. Both produce the same `ClassifiedTool` shape so downstream code is uniform. Every `ToolCallAction` carries:
- `tool_type`: `"mcp" | "skill" | "sub_agent" | null` — coarse bucket
- `tool_name`: canonical identity inside the bucket:
  - `mcp` → bare MCP tool name **without** the `mcp__<server>__` / `MCP:` prefix (e.g. `"bdt_navigation_go-to"`)
  - `skill` → skill name from `tool_input.skill` (e.g. `"commit"`)
  - `sub_agent` → sub-agent type from `tool_input.subagent_type` (e.g. `"code-reviewer"`)
  - `null` → raw native tool_name (`"Read"`, `"Bash"`, `"Edit"`, …)
- `mcp_server`: populated only when `tool_type="mcp"`. Claude derives it from its `mcp__<server>__<tool>` naming; Cursor's wire format has no server segment so the classifier returns `null`, and the Cursor track-action adapter overrides via prefix lookup — `bdt_*` → `"browser-devtools"`, `ndt_*` → `"node-devtools"`. Other unknown MCP tools in Cursor stay `null`.

`verify-gate` filters per active cycle: browser cycle compares `tool_type === "mcp" && mcp_server === "browser-devtools"` against `browser.alwaysRequired` + `evidencePaths`; node cycle compares `mcp_server === "node-devtools"` against `backend.node.alwaysRequired` + `evidencePaths`. Multi-cycle gates run both checks in parallel, AND-combined for the final result.

## Session Actions (`actions.jsonl`)
All session events are recorded in `.ironbee/sessions/<session_id>/actions.jsonl`:

Hierarchy: `session_id` → `activity_id` → `verification_id` + `fix_id`. Child events carry their parent IDs so downstream analytics / collector can group and correlate events.

| Type | Recorded By | Purpose | Parent IDs |
|---|---|---|---|
| `session_start` | SessionStart hook | Session lifecycle tracking | — |
| `session_end` | SessionEnd hook | Session end with reason and duration | — |
| `activity_start` | activity-start / PreToolUse (fallback) | Agent turn begins (source: user_prompt or pre_tool_use) | `activity_id` (new) |
| `activity_end` | verify-gate (Stop) | Agent turn ends (includes duration_ms) | `activity_id` |
| `verification_start` | verification-start | Opens verification cycle | `activity_id`, `verification_id` (new), `trace_id` (new) |
| `verification_end` | submit-verdict (auto) | Closes verification cycle | `activity_id`, `verification_id`, `trace_id` |
| `verdict_write` | submit-verdict | Records verdict content | `activity_id`, `verification_id`, `trace_id` |
| `fix_start` | submit-verdict (on fail) | Opens fix cycle, sets phase to "fixing" | `activity_id`, `fix_id` (new) |
| `fix_end` | verification-start (if fixing) | Closes fix cycle, transitions to "verifying" | `activity_id`, `fix_id` |
| `file_change` | clear-verdict hook | Tracks code file changes (Write/Edit/StrReplace/Delete, filtered by verifyPatterns); carries `operation: "create" \| "update" \| "delete"` plus `lines_added` / `lines_removed` (null when unknown) | `activity_id`, `fix_id` |
| `tool_call` | track-action hook | Tracks devtools MCP tool usage (`bdt_*` for browser cycle, `ndt_*` for node cycle). Other (non-devtools) tools route to the queue, not actions.jsonl. | `activity_id`, `verification_id`, `trace_id?` |
| `verification_requested` | verify-gate | Marker with action (allow/block) and reason | `activity_id` |

## Runtime Files (session-isolated, gitignored)
```
.ironbee/sessions/<session_id>/
  actions.jsonl    Session event log (append-only)
  verdict.json     Current verdict (cleared on code edit)
  state.json       Session state (retries, activeVerificationId, activeTraceId, lastVerdictStatus, activeFixId, activeActivityId, phase, recordingRequired, recordingActive, active)
  session.log      Debug/error log
  queue/           Queue module artifacts (see Queue Module section)
```

## Queue Module (`src/queue/`)
File-backed, crash-safe, concurrency-safe background job queue. Canonical spec: **`docs/job-queue.md`** — when in doubt, the spec is authoritative.

Producers append jobs to `jobs.jsonl` via atomic `O_APPEND` writes (4 KB line limit). A `snapshot()` atomic-renames the live file into `jobs-<UUID>.jsonl`, which a one-shot `ironbee process-job-file <path>` worker consumes. Workers never share files — each snapshot is owned by exactly one worker.

### Public API (`import { … } from "../queue"`)
```
submit(projectDir, sessionId, type, data, opts?)   → job_id          §5.1
snapshot(projectDir, sessionId)                    → path | null     §5.2
processFile(path, opts?)                           → void            §5.3
drain(projectDir, sessionId, opts?)                → DrainResult     §5.4
register(type, handler) / unregister(type)         handler registry
spawnDetachedWorker(snapshotPath)                  detached spawn    §9.1
flushInBackground(projectDir, sessionId)           Stop-hook flush   §5.6
flushSynchronously(projectDir, sessionId, opts?)   SessionEnd flush  §5.6
maybeAutoFlush(projectDir, sessionId)              size-based flush  §5.6
```

### Error classes (handler contract)
| Throw from handler | Worker action |
|---|---|
| `TransientError` | leave snapshot; retried on next drain |
| `AuthError` | leave snapshot; log ERROR for operator |
| `BadRequestBatchError` (from `dispatch`) | fall back to per-job `dispatchSingle` |
| `BadRequestError` (from `dispatchSingle`) | dead-letter that job; continue |

Plus queue-internal: `InvalidSessionIdError`, `JobTooLargeError`, `IOError`.

### Runtime layout (`<projectDir>/.ironbee/sessions/<sid>/queue/`)
| File | Owner | Purpose |
|---|---|---|
| `jobs.jsonl` | producers | Live append-only file (sub-ms `submit()`) |
| `jobs-<UUID>.jsonl` | worker | Frozen snapshot being / awaiting processing |
| `dead-letter.jsonl` | worker | Permanent per-line failures (rotates at 10MB / 10k lines, retains 3 newest by mtime) |
| `dead-letter-<UUID>.jsonl` | worker | Rotated dead-letter files |
| `worker.log` | worker | Append-only worker log (rotates at 10MB, retains 5 newest by mtime) |
| `worker-<UUID>.log` | worker | Rotated worker logs |

Queue **never** touches the parent session dir (`actions.jsonl`, `verdict.json`, `state.json`, `session.log` are consumer-owned).

### Master switch
The queue is **opt-in by config presence**:
- No `jobQueue` section → queue is disabled. `submit()` is a silent no-op (debug-logged) and the track-action adapter skips wire construction altogether.
- `jobQueue` section present (even empty `{}`) → queue is enabled. Adding any sub-key (e.g. just `autoFlushSizeBytes`) implicitly turns it on.
- `jobQueue.enable: false` (explicit) → queue is disabled, even if other `jobQueue` keys are set. Lets you keep a tuned config around without running the pipeline.

Operator commands that submit (e.g. `ironbee queue dead-letter retry`) fail loudly with a clear error message when the queue is disabled, instead of silently dropping the resubmit.

### Sync vs async (§5.6)
- **During active use** (per-turn hook, mid-session trigger): call `snapshot()` → `spawnDetachedWorker()`. Trigger returns in <10 ms.
- **At session shutdown**: call `drain()` synchronously; blocks a few ms to low seconds.
- **Operator recovery**: `ironbee queue drain [--session <id>]` — synchronous per-session final-flush.
- **Size-based auto-flush (producer-side)**: `submit()` invokes `maybeAutoFlush()` after every successful append. When the live `jobs.jsonl` size meets or exceeds `config.jobQueue.autoFlushSizeBytes` (default **32 KB**), a detached worker is spawned mid-session — same mechanism as Stop hook, just driven by queue growth. Set the threshold to `0` or a negative value to disable (lifecycle flushes remain).
- **Interval-based auto-flush (producer-side)**: same `maybeAutoFlush()` call also checks `now − birthtime(jobs.jsonl)` against `config.jobQueue.autoFlushIntervalSeconds` (default **60 s**). `birthtime` (the file's creation time, set on the next submit after a snapshot) is the timestamp of the **first event in the current batch**, so this measures "events should not sit longer than X seconds before reaching the collector." Covers low-tool / long-thinking turns where size never trips. Whichever check fires first wins; both `0` / negative disables the producer-side path entirely. Note: requires a filesystem with reliable `birthtime` (macOS APFS, Linux ext4 ≥ 4.11, NTFS) — older Linux silently falls back to size + Stop only.

### Flush wiring (already in place)
| Hook | Helper | Mode |
|---|---|---|
| Stop (Claude `verify-gate`, Cursor `stop`) | `flushInBackground(projectDir, sessionId)` | `snapshot()` + detached worker, fail-safe |
| SessionEnd (Claude `session-end`, Cursor `sessionEnd`) | `await flushSynchronously(projectDir, sessionId)` | `drain()`, fail-safe |

Both helpers are no-ops when there's nothing to flush, so until a producer is wired they cost one `fs.stat`/`rename` attempt.

### Operator commands
`ironbee queue status|drain|dead-letter {list,stats,retry,clear}|purge` — all scoped to the current project, aggregate across sessions by default, narrow with `--session <id>`.

### First consumer — `send_event` (tool_call → collector)
Each PostToolUse **and** PostToolUseFailure fires `track-action`, which **every tool** triggers. Split routing:

- **devtools MCP tools** (classified as `tool_type="mcp"` + `mcp_server in {"browser-devtools", "node-devtools"}`): recorded in `actions.jsonl` (verify-gate dep). Browser-cycle tools (`bdt_*`) update recording state; node-cycle tools (`ndt_*`) never touch recording flags (browser-only enforcement, §12 of design doc). **NOT** submitted to the queue — both servers are the same npm package and self-ship their `tool_call` events to the collector; sending them again would duplicate without a dedup key. Nested `callTool()` extracts (from `bdt_execute`) follow the same rule.
- **All other tools** (Read/Write/Bash/…, including non-browser MCP, skills, and sub-agents): submit a `send_event` job.

For the queue wire payload we apply two different policies to the two raw fields:

- **`tool_input`** — passed through a per-tool whitelist (`src/clients/claude/util.ts:extractClaudeToolInput`) so the collector sees useful labels (`file_path`, `pattern`, `url`, `subagent_type`, …) but never raw content. Tool list + per-tool key sets come from `docs/claude-builtin-tool-schemas.md` — single source of truth. Coverage spans all 17 documented Claude Code built-ins (Bash / BashOutput / KillShell / Read / Write / Edit / NotebookEdit / Glob / Grep / WebFetch / WebSearch / Agent / Task / TodoWrite / ExitPlanMode / AskUserQuestion / Skill / SlashCommand). Drops raw content (`Write.content`, `Edit.old_string` / `new_string`, `NotebookEdit.new_source`, full `Bash.command` text — only the binary name passes through). Drops user-supplied argument tails on `Skill.args` and the trailing args on `SlashCommand.command`. Keeps high-signal agent-authored free text where it parallels Cursor (`Agent.prompt`, `WebFetch.prompt`, `WebSearch.query`, `TodoWrite.todos`, `ExitPlanMode.plan`, `AskUserQuestion.questions`) — collector consumers should be aware these can carry project context. Unknown / non-built-in tools → `undefined`. Browser-devtools (`mcp__browser-devtools__*`) tools bypass extraction entirely — kept as-is (verify-gate + `bdt_execute` nested-callTool parser need raw shape; these never reach the queue anyway).
- **`tool_response`** — fully stripped from the wire (`Bash` stdout, `Read` file bytes, API responses are too varied / risky to whitelist). Only its byte size goes through.

`tool_input_size` and `tool_response_size` reflect the **original raw byte counts** so analytics can still see "agent wrote a 50KB file via Write" even when only `file_path` made it onto the wire. `actions.jsonl` carries the same projected `tool_input` for native tools; for browser-devtools tools it keeps the raw input (used by the local nested-callTool parser).

Failure events carry `error` and omit response; nested extraction is skipped for failed `bdt_execute` calls. `duration` is read verbatim from Claude's PostToolUse `duration_ms` (pure tool execution, excluding PreToolUse hooks + permission-prompt wait); Cursor supplies it on postToolUseFailure the same way; `null` only on pre-duration-support Claude versions.

Cursor has its own per-tool whitelist (`src/clients/cursor/util.ts:extractCursorToolInput`) covering all 18 built-in tools per the schemas captured in `docs/cursor-builtin-tool-schemas.md`. Same risk model as Claude: keep coarse labels (paths, patterns, mode flags), drop raw content (file bodies via `Write.contents`, `EditNotebook` cell bodies, `StrReplace` before/after strings, full `Shell.command` text — only the binary name passes through). Mirrors Claude's whitelist decisions where they apply: `Task.{description, prompt, subagent_type, model}` parallels Claude `Agent`; `AskQuestion.{title, questions}` parallels Claude `AskUserQuestion`. Note Cursor's own field names differ from Claude's in several cases — `Read`/`Write`/`Delete`/`StrReplace` use `path` (not `file_path`); `Write` uses `contents` (not `content`); `WebSearch` uses `search_term` (not `query`); `Glob` uses `glob_pattern` + `target_directory`.

A later Stop hook's `flushInBackground` (or operator `ironbee queue drain`) spawns a worker that dispatches the batch. The `send_event` handler chunks the batch into `config.collector.batchSize` (default 100) events per POST to `{collector.url}/v1/events`.

| Tool | actions.jsonl | Queue (`send_event`) |
|---|---|---|
| browser-devtools (`tool_type="mcp"` + `mcp_server="browser-devtools"`) | ✅ (verify-gate dep) | ❌ (mcp ships it itself) |
| node-devtools (`tool_type="mcp"` + `mcp_server="node-devtools"`) | ✅ (verify-gate dep) | ❌ (same package, ships it itself) |
| All other tools (Read/Write/Bash/other MCP/skills/sub-agents) | ❌ | ✅ |

- Handler: `src/queue/handlers/send-event.ts` — HTTP status → TransientError / AuthError / BadRequestBatchError / BadRequestError per spec §6. No-ops when `collector` isn't configured or `IRONBEE_COLLECTOR=false`.
- Registration: `src/queue/register-handlers.ts` → `register(SEND_EVENT_TYPE, sendEventHandler)`, called from `src/index.ts` at startup.
- Oversized events: when `tool_response` is huge (big Bash output, big Read), submit retries without it and adds `tool_response_omitted: true`. If still too large the event is dropped with a debug log.

## Collector (`collector.ts`)
Sends session action events to a remote IronBee Platform collector endpoint.
- Configured via `collector.url` and `collector.apiKey` in `config.json`. The collector is opt-in by `collector` section presence (mirrors `jobQueue`); set `collector.enable: false` to suspend without removing the rest of the config.
- Called from `appendAction()` on every action write **except `tool_call`** (session_start, session_end, activity_start/end, file_change, verification_start/end, verdict_write, fix_start/end, verification_requested)
- `tool_call` events are sent by browser-devtools-mcp directly (via `_metadata.collectorUrl` / `_metadata.collectorApiKey` passed in PreToolUse)
- Async with 3s timeout, fail-safe (never throws, never blocks CLI)
- Posts JSON array of events to `{collector.url}/v1/events` with `X-API-Key` header
- When the `collector` section is absent, `enable: false`, or `url` is empty, the client is a no-op (local-only mode)
- `_metadata` is stripped from `tool_input` before writing to `actions.jsonl`
- All hook functions and `appendAction` are async to ensure collector events are sent before process exit
- Payload is passed through as-is — `id`, `session_id`, `project_name`, and epoch-ms `timestamp` are already populated by `appendAction`/`baseFields`; no wire-time conversion needed

## Tech Stack
- TypeScript → CommonJS (`dist/`)
- `commander` — CLI framework
- `jest` + `ts-jest` — tests
- `eslint` v9 (flat config) + `@typescript-eslint`

## Scripts
```
npm run build       tsc + copy .md/.mdc files to dist/
npm run dev         ts-node src/index.ts
npm run lint        eslint .
npm run lint:fix    eslint . --fix
npm run test        jest
npm run clean       rm -rf dist
```

---

## Rules for Claude

### Code Style
- **Always** add explicit type annotations on every constant, variable, function parameter, and return type
- **Always** use curly braces `{}` for `if/else/for/while` — no braceless single-line forms
- These are enforced by ESLint — run `npm run lint` before considering a task done

### Workflow
- After writing or editing TypeScript files, always run `npm run lint && npm run build` to verify
- Skill/rule content lives in `src/clients/<client>/skills/` and `src/clients/<client>/rules/` — copied to `dist/` at build time, read via `__dirname` at runtime
- Per-runtime guidance for skill/rule/command md files lives in `src/clients/<client>/fragments/<base>.<runtime>[.<key>].md` (always `.md`, even when the target is `.mdc`). When you add a new keyed marker block to a source md file, add the matching fragment file in BOTH client fragment dirs — missing fragments fall back to the placeholder. The list of files that carry markers is in `RUNTIME_TARGETS` in `src/lib/runtime-section.ts`.

### Testing
- When writing, updating, or deleting code, always update the corresponding unit and integration tests
- Unit tests: `tests/unit/` — core logic (actions, config, verify-gate, clear-verdict)
- Integration tests: `tests/integration/` — full hook chain with real filesystem
- Client-specific tests: `tests/clients/<client-name>/` — adapter tests (stdin parsing, exit codes)
- Run `npm test` after any change to ensure all tests pass

### Analytics
- Analytics metrics, formulas, and explanations are documented in both `README.md` (Analytics section) and `docs/intelligence-analysis.md`
- When changing analysis logic (scoring formulas, new metrics, label changes), update **all of**: README.md, docs/intelligence-analysis.md, llms.txt, and llms-full.txt
- Scoring formulas: Efficiency = `coding_time / (coding_time + fix_time) × 100`, Quality = average of 4 components (pass rate, page coverage, check depth, error cleanliness), Confidence = `pass_count / total_verdicts × 100`

### Flow doc maintenance (`docs/flow.md`)
- `docs/flow.md` is the canonical end-to-end runtime flow spec — hook fire order, per-hook allow/deny branches, verify-gate decision tree, cycle lifecycles, state machine, cleanup paths.
- **Whenever the flow changes, update `docs/flow.md` in the same change.** This includes:
  - Adding / removing / renaming a hook, or changing its matcher
  - Changing what a hook reads, writes, or its allow/deny branches
  - Adding a new verify-gate check, reason string, or block/allow path
  - New cycle types or changes to existing cycle lifecycles (`activity` / `verification` / `fix`)
  - New `state.json` fields or transitions
  - New cleanup paths (`closeOpenCycles`, `reconcile*`)
  - Changes to multi-cycle activation, satisfier, evidence shape, or pass-override logic
- If a code change makes `flow.md` stale, update it in the same PR. Stale flow docs are worse than no docs.
- The `CLAUDE.md` "Verification Flow" / "Activity Tracking" / "Recording Enforcement" sections are summaries — keep them in sync with `flow.md` when the underlying behavior changes.

### CLAUDE.md Maintenance
- When new commands, architectural decisions, or conventions are established, update this file
- Keep the Rules section current — add new rules here as they are agreed upon in conversation

---
> Source: [ironbee-ai/ironbee-cli](https://github.com/ironbee-ai/ironbee-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
