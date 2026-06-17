## omnisess

> |


# omnisess

Aggregate AI coding sessions across Claude Code, Cursor, Codex, and GitHub Copilot CLI — search, list, and detect active sessions from one place.

## Prerequisites

The `omnisess` binary must be installed and in your PATH:

```bash
go install github.com/psacc/omnisess@latest
```

## Skills

| Skill | Description |
|-------|-------------|
| `omnisess-search` | Full-text search across all sessions |
| `omnisess-list` | List sessions with filtering by tool, time, and limit |
| `omnisess-active` | Show currently running sessions |
| `omnisess-ps` | Process tree of live Claude and Codex sessions with ancestor lineage (macOS) |
| `omnisess-show` | Show full detail for a single session by ID |
| `omnisess-recap` | Structured markdown briefing of sessions for a time window |
| `omnisess-digest` | Print a calendar day's sessions as Obsidian-compatible markdown with full Q&A |
| `omnisess-stats` | Tool-call counts and file-I/O activity from the transcript index |
| `omnisess-index` | Bulk-populate the transcript SQLite cache used by `omnisess-stats` |
| `omnisess-skills-audit` | Classify Claude Code skills by usage (Keep / Borderline / Archive) |

## Supported Sources

| Source | Status |
|--------|--------|
| Claude Code | Full |
| Cursor | Full |
| Codex | Full |
| GitHub Copilot CLI | Full |

---
> Source: [psacc/omnisess](https://github.com/psacc/omnisess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
