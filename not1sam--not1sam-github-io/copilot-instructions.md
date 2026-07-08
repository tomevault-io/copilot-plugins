## not1sam-github-io

> This project uses a local Obsidian vault for persistent memory across sessions and AI tools.

# Memory Protocol — Infinite Memory for All AI Agents

This project uses a local Obsidian vault for persistent memory across sessions and AI tools.
Every task, fix, bug, decision, and change MUST be documented so that **any AI agent, on any machine, at any time** has full context.

## Memory location

```
./memory/
```

OpenCode loads these files automatically via `opencode.json`. Other tools must read them manually.

## Rules — Strict

### 1. START every session
Read ALL of these files before doing any work:
- `progress.md` — session history, what's been done, what's next
- `decisions.md` — architectural & design decisions with rationale
- `facts.md` — complete project encyclopedia (tree, every file, exports, how it works)
- `audit_plan.md` — living checklist of bugs to fix and features to add; **if you complete any item, mark it `[x]` and remove its description text**

### 2. END every task — document EVERYTHING
After completing any task (fix, feature, refactor, config change, anything):

**Update `progress.md`** — add a new entry at the top with:
- The **issue** or **problem** that triggered the task
- The **root cause** (if a bug)
- The **fix** or **change** that was applied
- Every **file modified** and what changed in it

**Update `decisions.md`** — if a new decision was made, add it with:
- The decision and why it was chosen over alternatives

**Create/update `sessions/YYYY-MM-DD.md`** — full session log:
- Every file touched and what changed
- Commands run
- Reasoning behind choices

### 3. Be explicit
Don't write "fixed a bug" — write:
> **Issue:** Typewriter animation stuttered on slow connections
> **Root cause:** Font loading delay caused layout shift before animation start
> **Fix:** Added font-display: swap and preconnect to Google Fonts
> **File:** `src/index.css:5`, `index.html:8`

### 4. Cross-tool portability
Plain markdown files. Any AI tool (Claude Code, Gemini CLI, Aider, Cursor, opencode, etc.) can read and write them. No proprietary format, no MCP dependency.

### 5. Git-free zone
The vault is **NOT** inside the project repo (`../obsidian-vaults/`). Memory stays local and is never committed. Changes to memory files do NOT go through git.

### 6. Facts.md is the single source of truth
Keep `facts.md` up to date with:
- New files added or removed
- New routes, components, or pages
- Changes to theming, CSS variables, or layout
- Changes to deployment or CI/CD
- New environment variables
- GitHub API usage changes

## Summary — Do this every task

```
1. Read progress.md + decisions.md + facts.md + audit_plan.md
2. Do the work
3. Mark any completed audit items as [x] in audit_plan.md and remove their description text
4. Update progress.md (issue, root cause, fix, files changed)
5. Update decisions.md (if applicable)
6. Update sessions/YYYY-MM-DD.md (full log)
7. Update facts.md (if project structure changed)
```

---
> Source: [Not1Sam/Not1Sam.github.io](https://github.com/Not1Sam/Not1Sam.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
