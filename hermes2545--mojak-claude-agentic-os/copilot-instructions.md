## mojak-claude-agentic-os

> This directory is the user's Agentic OS — a plain-markdown operating system

# AGENTS.md — the map

## Identity & mission
This directory is the user's Agentic OS — a plain-markdown operating system
for tasks, goals, knowledge, and memory, operated through Claude Code.

## Map
| Path | Purpose |
|---|---|
| `CLAUDE.md` | Thin wrapper, loads this file |
| `AGENTS.md` | This map (instructions layer) |
| `GOALS.md` | Priority layer — Now/Next/Someday |
| `BACKLOG.md` | Inbox — brain-dump, converted via /backlog |
| `tasks/` | State layer — one file per task, `_template.md` |
| `tasks/archive/` | Completed tasks, moved by /done |
| `memory/working/` | Scratch, current-focus notes (volatile) |
| `memory/episodic/` | Session notes + tool logs (`logs/`, gitignored) |
| `memory/semantic/` | Durable graduated lessons (append-only) |
| `memory/personal/` | User profile, edited deliberately |
| `memory/.staging/` | Lesson candidates awaiting /review-memory |
| `knowledge/` | Reference notes, saved research |
| `docs/` | On-demand harness docs (this file links to them) |
| `protocols/permissions.md` | DENY rules enforced by the PreToolUse hook |
| `system/hooks/` | `pre_tool_guard.py`, `post_tool_log.py`, `agent_event_log.py` |
| `system/scripts/` | `dream.py`, `graduate.py`, `reject.py`, `recall.py`, `install-cron.sh`, `asana_scan.py`, `backup.sh` |
| `system/templates/` | Fresh-clone seeds: `session-note.md`, `GOALS.md`, `BACKLOG.md`, `profile.md` |
| `system/google/` | OS Google integration (your-google-account): Drive upload, Gmail send/read, Calendar, Docs/Sheets/Slides — see `docs/google-setup.md` |
| `system/asana/` | Asana integration (task scan/sync) — see `docs/asana-setup.md` |
| `system/telegram/` | Telegram bot bridge — see `docs/telegram-setup.md` |
| `webui/` | Browser UI: server (Agent SDK) + React app + in-browser terminal — see `webui/README.md`, `docs/webui-plan.md` |
| `.claude/agents/` | coder, researcher, librarian, planner, reviewer |
| `.claude/skills/` | daily, backlog, remember, recall, review-memory, done, weekly, build, os-audit |
| `MANUAL.md` | Thai user manual |

## Layers
- **Instructions** = `AGENTS.md` (this file)
- **Priority** = `GOALS.md`
- **Inbox** = `BACKLOG.md`
- **State** = `tasks/`
- **Memory** = `memory/` — 4 layers:
  - working: scratch / current focus, volatile
  - episodic: what happened — session notes + tool logs
  - semantic: durable, graduated lessons
  - personal: user profile and preferences
- **Reference** = `knowledge/`, `docs/`

## Operating rules
- Read `GOALS.md` before prioritizing any work.
- Convert backlog items into `tasks/` files using `tasks/_template.md`.
- Never claim completion without fresh evidence — see `docs/verification.md`.
- Write durable lessons as `LESSON:` lines in `memory/working/`, a session
  note, or a `tasks/` file; `dream.py` stages them, `/review-memory`
  graduates them.
- END OF EVERY SUBSTANTIAL SESSION: write a session note into
  `memory/episodic/` (from `system/templates/session-note.md`) with one
  `LESSON:` line per durable insight learned. Without this the memory
  pipeline starves — nothing reaches `.staging/` or `semantic/`.
- Do not edit `memory/semantic/` directly (hook-enforced) — use
  `system/scripts/graduate.py`.
- Prefer delegating code-writing to the `coder` subagent and research to
  `researcher` (cost tiering — see `docs/model-strategy.md`).
- Google delivery uses THIS OS's own account (your-google-account) via
  `system/scripts/deliver.sh` (Drive) and `system/google/` (Gmail, Calendar).
  NEVER use the `claude.ai` Google Drive/Gmail/Calendar connectors — they
  point at a different account. See `docs/google-setup.md`.
- Finish the whole job in ONE turn: when you delegate to a subagent whose
  result you still need (research → then write a file, etc.), run it
  SYNCHRONOUSLY and wait for the result before continuing. Do NOT launch a
  background subagent and end your turn saying you'll "follow up" — a
  one-shot run (e.g. the WebUI) has no next turn, so the run just ends and
  the result is lost. Only background a subagent if its result is truly
  fire-and-forget.

## On-demand docs
- Read `docs/golden-principles.md` for the 5 core principles behind this OS.
- Read `docs/memory-system.md` before touching anything under `memory/`.
- Read `docs/task-workflow.md` before creating or closing a task.
- Read `docs/verification.md` before marking anything done.
- Read `docs/model-strategy.md` before choosing a model or delegating.
- Read `docs/hooks-and-safety.md` before editing hooks or permission rules.

## Skills
| Skill | Purpose |
|---|---|
| `/daily` | Propose today's top 3 tasks from GOALS.md + open tasks/ |
| `/backlog` | Convert BACKLOG.md items into tasks/ files |
| `/remember` | Classify and save a fact into the right memory layer |
| `/recall` | Search memory/ + knowledge/ for relevant context |
| `/review-memory` | Graduate or reject staged lesson candidates |
| `/done` | Verification-first task completion and archival |
| `/weekly` | Weekly review: goals, dream.py, memory review, cleanup |
| `/build` | Cost-tiered plan → delegate → review → done loop |
| `/os-audit` | Read-only drift/freshness/organization audit of the OS; report saved to `audits/os-audit-YYYY-MM-DD.md` |

---
> Source: [hermes2545/mojak-claude-agentic-os](https://github.com/hermes2545/mojak-claude-agentic-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
