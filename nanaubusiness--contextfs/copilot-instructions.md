## contextfs

> Note: The `/contextfs` commands below are Claude Code only.

# ContextFS

Note: The `/contextfs` commands below are Claude Code only.
For Cursor, Codex, and other editors: use the `contextfs` CLI directly
(e.g. `contextfs build`, `contextfs query`) or rely on the background
watcher that auto-updates summaries on file save.

ContextFS generates plain-text summaries of your codebase. Claude reads summaries instead of raw files — same understanding, ~81% fewer tokens.

## Commands

### /contextfs build
Summarize every code file in the current project using Claude Opus 4.7.

```
/contextfs build
```

Run once after installing in a project. The FileChanged hook keeps summaries up to date automatically.

### /contextfs init
Set up ContextFS in a new project — installs the Claude Code hook and adds rules to CLAUDE.md.

```
/contextfs init
```

Run this once in each project you want ContextFS active in.

### /contextfs query
Find files related to a topic by searching their summaries.

```
/contextfs query auth
```

Returns ranked results with file paths and relevance scores.

### /contextfs demo
Preview what a summary looks like for any single file.

```
/contextfs demo src/utils/auth.ts
```

---

## Rule — Always Query Before Reading Raw Files

Before reading any source file, do this:

1. Run `/contextfs query <your-topic> --root .`
2. Read the `.summary` files returned
3. Only read the raw file if the summary is insufficient or missing

This applies to every file in every project.

---

## Summary Format

Each `.summary` file is plain text:

```
Purpose: Handles user authentication and session management
Exports: login, logout, verifyToken, refreshSession
Dependencies: bcrypt, jsonwebtoken, ./db/user.repository
Core logic:
  - login
  - logout
  - verifyToken
  - refreshToken
Risk: high
hash: abc123def456
```

---

## How It Works

1. **`/contextfs build`** — Claude Opus 4.7 summarizes all code files, writes `*.summary` sidecar files and `context-map.json`
2. **Background watcher** — Every file save triggers `contextfs build --target <file>` automatically (all editors)
3. **`/contextfs query`** — Searches summaries by relevance, returns top matches with scores
4. **Claude Code (MCP)** — The MCP server intercepts file reads and returns `.summary` content automatically when available. Files without summaries trigger an approval prompt.
5. **Other editors** — Summaries update on file save via the watcher. Use `contextfs query` CLI to search summaries before reading raw files.

---
> Source: [nanaubusiness/contextfs](https://github.com/nanaubusiness/contextfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
