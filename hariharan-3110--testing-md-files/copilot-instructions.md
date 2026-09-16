## testing-md-files

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A minimal Vite + React scaffold used to demo a single react-bootstrap page: a table with a "Create data" button that appends a mock row (name + role) client-side. There is no backend, no routing, and no state persistence — this is throwaway/test UI, not a product codebase. Keep additions equally minimal unless told otherwise.

## Commands

- `npm run dev` — start the Vite dev server (http://localhost:5173/)
- `npm run build` — production build
- `npm run preview` — preview the production build
- `npm run lint` — run oxlint (config in `.oxlintrc.json`)

There is no test runner configured in this project.

### npm registry note

The global npm config on this machine points at a private CodeArtifact registry whose saved auth token is expired, so plain `npm install` will 401. Use the public registry explicitly, plus `--legacy-peer-deps` if installing at the workspace root of the sibling `react-bootstrap` clone (unrelated peer-dep conflict there):

```
npm install --registry=https://registry.npmjs.org/
```

## Architecture

Everything lives in `src/`:
- `main.jsx` — entry point; imports Bootstrap's CSS (`bootstrap/dist/css/bootstrap.min.css`) globally before mounting `App`.
- `App.jsx` — the entire UI. Local `useState` array of rows; `Create data` button pushes a randomly generated row (`randomRow`) picking from fixed `NAMES`/`ROLES` arrays. No props drilling, no additional components — keep it a single file unless the scope grows.

Uses `react-bootstrap` components directly (`Container`, `Table`, `Button`) imported individually from `react-bootstrap/<Component>` rather than the aggregate package import.

## Claude Code extension points (team reference)

This repo also demonstrates the four ways Claude Code is extended per-project. Each is checked into version control here so the whole team gets the same behavior.

For a full walkthrough with actual captured output from real runs (not hypothetical examples), see [docs/claude-code-live-demo.md](docs/claude-code-live-demo.md).

### 1. CLAUDE.md (this file)
Always-loaded project context — commands, architecture, conventions. Claude reads it automatically at the start of every session in this directory. Use it for things a new contributor (human or Claude) would otherwise have to rediscover by reading multiple files.

### 2. Skills — `.claude/skills/<name>/SKILL.md`
A packaged, reusable procedure Claude follows for a specific recurring task. Unlike CLAUDE.md (always loaded), a skill is invoked — either automatically when its `description` matches the task, or explicitly via `/<name>`.

Example here: [.claude/skills/add-table-field/SKILL.md](.claude/skills/add-table-field/SKILL.md) — encodes the exact steps to add a new column to the mock table (add a sample-value array, extend `randomRow`, add the `<th>`/`<td>`). Live demo: ask Claude "add an email column to the table" and watch it follow the skill's steps instead of improvising a different structure each time.

### 3. Subagents — `.claude/agents/<name>.md`
A separate Claude instance with its own system prompt, restricted tool access, and (optionally) a different model — used to delegate a bounded task without polluting the main conversation's context.

Example here: [.claude/agents/code-reviewer.md](.claude/agents/code-reviewer.md) — read-only (no `edit`/`write` tools), scoped to reviewing this repo's React changes against its "stay minimal, no backend" constraints. Live demo: ask Claude to "use the code-reviewer subagent to review App.jsx."

### 4. MCP servers — `.mcp.json`
Configuration for external Model Context Protocol servers that give Claude additional tools (e.g. filesystem access beyond the project, a database, an API). Declared per-project so the whole team connects to the same tools.

Example here: [.mcp.json](.mcp.json) — a filesystem server example, included for illustration only (not connected in this demo). Team members would run `claude mcp` or restart Claude Code after editing `.mcp.json` to actually activate a server; explain this is config, not code Claude executes directly.

### Memory (per-user — NOT the shared-learning mechanism)
Claude Code separately keeps a persistent, per-user, per-machine memory across sessions, stored outside this repo (on this machine: `~/.claude/projects/<project-id>/memory/`, an `MEMORY.md` index plus one file per memory). **This is easy to mistake for a team-shared "lessons learned" log — it is not.** It's not checked into git and one developer's memory is invisible to everyone else's Claude sessions. It exists to remember things about *you* (your preferences, how you like to work) across all your projects, not to record project knowledge for the team.

It's populated selectively: when you explicitly say "remember this," when you correct Claude's approach, when you confirm a non-obvious approach worked, or when a durable fact comes up that isn't derivable by reading the code. A fresh repo with routine conversation may have an empty memory directory — that's expected, not a bug.

**For anything the whole team should benefit from — a mistake made and fixed, a gotcha, a convention settled on — put it in this file (CLAUDE.md) and commit it, not in memory.** See "Lessons learned" below for the actual shared mechanism.

### Lessons learned (shared, git-tracked)

This is the pattern for team-shared knowledge: when a mistake or gotcha gets hit and resolved, it gets written here and committed, so every developer's Claude session — and every human on the team — has it from that point on. Real example from building this repo:

- **npm install 401 errors on this machine.** The global npm registry is set to a private CodeArtifact repo whose saved auth token had expired, so any `npm install` without an explicit registry override failed with `E401`. Fix: pass `--registry=https://registry.npmjs.org/` explicitly (see "npm registry note" above). This is now documented here instead of every developer independently rediscovering and re-debugging the same 401.

Add new entries above this line as they come up — don't let them live only in chat history or a memory file only one person can see.

---
> Source: [Hariharan-3110/testing-md-files](https://github.com/Hariharan-3110/testing-md-files) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
