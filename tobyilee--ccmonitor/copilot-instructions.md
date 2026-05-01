## ccmonitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ccmonitor** — A standalone Bun TUI app that monitors Claude Code session activity in real time. It watches `~/.claude/` data files (JSONL transcripts, subagent metadata, team configs, task files) and renders a terminal dashboard showing tool usage, skills, subagents, teams, tasks, token consumption, and file activity.

## Build & Run Commands

```bash
bun run start          # Run the monitor (bun run src/index.ts)
bun run dev            # Run with --watch for auto-reload
bun run build          # Compile to standalone binary: dist/ccmonitor
bun run install:global # Build + copy binary to ~/.bun/bin/ccmonitor
```

Run with a specific session: `bun run src/index.ts <sessionId>`

## Testing

No test suite exists yet (`bun test` is not wired up). Verification is manual:
1. `bun run build` must succeed.
2. `bun run start` must render visually-identical TUI output to the prior version.
3. A `bun test` setup is planned — see `tasks/prd-quality-uplift.md` US-002 / US-005.

## Upgrade

When the user says `upgrade`:
- Bump the minor version in `package.json`
- Verify with `bun run build` (rebuild `dist/ccmonitor`)
- Add an entry to `CHANGELOG.md` (Keep a Changelog format)
- Update `README.md` (version line + any user-facing changes)
- Save, commit, push

## Architecture

Four source files in `src/`, no external UI framework — raw ANSI escape codes to stdout:

- **index.ts** — Entry point. Sets up chokidar file watcher on `~/.claude/` paths, manages keyboard input (q/r), runs a 2-second `setTimeout` refresh loop calling `refreshState()` → `render()`. Maintains a ring buffer of file events (max 50).
- **parser.ts** — All data extraction logic. `findLatestSession(cwd)` locates the most recent `.jsonl` transcript for a project. `parseTranscript()` reads the JSONL line-by-line and builds `SessionState` by processing each entry for: tool usage counts, skill invocations (both slash-command `<command-name>` tags and `Skill` tool_use blocks), token/message counters, model info, team references. After transcript parsing, loads subagent metadata, team configs, tasks, and hook state from disk.
- **types.ts** — All TypeScript interfaces. `SessionState` is the central data structure; `TranscriptEntry` models the JSONL schema.
- **ui.ts** — Pure rendering. Builds ANSI box-drawing output from `SessionState`. Handles CJK fullwidth character widths for correct terminal alignment. No state mutation.

### Key Design Decisions

- **CWD-scoped by default**: Without a sessionId argument, the monitor finds the latest session matching `process.cwd()` by converting the path to Claude's directory naming convention (`/Users/foo/bar` → `-Users-foo-bar`).
  - **Known footgun:** `cwdToProjectDirName` maps `_` → `-` (fixed in PR #1), but the inverse `projectDirToRealCwd` only maps `-` → `/`. A cwd containing `_` (e.g. `/Users/foo_bar/baz`) round-trips lossy. Tracked in `quality-20260417.md` Issue #4 and `tasks/prd-quality-uplift.md` NG-6.
- **Subagent status heuristic**: A subagent is considered "completed" if its `.jsonl` file hasn't been modified in 30 seconds.
- **Skill completion detection**: Two mechanisms — (1) transcript parsing detects `stop_reason === 'end_turn'` on assistant messages, (2) a PostToolUse hook (`scripts/on-skill-complete.sh`) writes to `~/.claude/.omc/state/last-skill-complete.json` for faster detection.
- **No imports from Claude Code codebase** — fully decoupled. Reads only the file artifacts Claude Code produces.

### Reference: `~/.claude/` File Structure

The monitor reads these paths (documented in detail in `filegen.md`):

| Path Pattern | Content |
|---|---|
| `projects/<sanitized-cwd>/<sessionId>.jsonl` | Session transcript (JSONL, append-only) |
| `projects/<sanitized-cwd>/<sessionId>/subagents/agent-*.meta.json` | Subagent metadata |
| `projects/<sanitized-cwd>/<sessionId>/subagents/agent-*.jsonl` | Subagent transcripts |
| `teams/<name>/config.json` | Team member list |
| `teams/<name>/inboxes/` | Team message inboxes |
| `tasks/<sessionId>/*.json` | Task/TODO data |
| `sessions/<pid>.json` | Active session registry |
| `.omc/state/last-skill-complete.json` | Hook-written skill completion signal |

### Hook Script

`scripts/on-skill-complete.sh` — A PostToolUse hook for the Skill tool. Receives `PostToolUseHookInput` JSON on stdin, extracts skill name/args, writes completion state JSON. Install by referencing it in Claude Code's `settings.json` hooks config.

---
> Source: [tobyilee/ccmonitor](https://github.com/tobyilee/ccmonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
