## myrlin-workbook

> **You MUST only create, edit, and modify files within this project directory.**

# Claude Workspace Manager

## CRITICAL SCOPE CONSTRAINT
**You MUST only create, edit, and modify files within this project directory.**

**NEVER modify files outside this folder.** This includes:
- Do NOT edit `~/.claude/settings.json` or any global config
- Do NOT edit files in other projects
- Do NOT modify system files
- All scripts, configs, tests, and output MUST stay within this folder

If you need to READ files outside this folder (e.g., to understand Claude session data), that's fine. But all WRITES stay here.

## Project Goal
Build a lightweight terminal workspace manager for Claude Code sessions that:
1. **Persists session state** - tracks which Claude sessions are running, their IDs, working directories, topics
2. **Auto-recovers after crash/restart** - saves state to disk, auto-reopens sessions on launch
3. **Groups sessions** - organize related sessions into named workspaces
4. **Notifications** - surface agent status, completion, errors
5. **Terminal UI** - clean, fast, minimal. Think tmux-inspired but purpose-built for Claude sessions

## Tech Stack Preferences
- PowerShell or Node.js (must work natively on Windows without WSL)
- State persisted to JSON in this folder
- Leverage Windows Terminal tabs/panes if possible
- Keep dependencies minimal

## Agent Teams
This project has agent teams enabled. Use teammates for:
- One teammate for core state management logic
- One teammate for terminal UI/display
- One teammate for testing and screenshots
- Coordinate via the lead agent

## Testing
- Build tests alongside the code
- Take screenshots of the working UI using Playwright or similar
- Store screenshots in `./screenshots/`

## Feature Session Workflow (NON-NEGOTIABLE)

**Every non-trivial feature MUST be built in a dedicated Claude Code session on its own git branch/worktree.**

### Process

1. **Create a git worktree** for the feature: `git worktree add -b feat/<feature-name> ../cwm-feat-<feature-name>`
2. **Spawn a new Claude Code session** pointed at the worktree directory
3. **Feed it ALL relevant context** — point it at every file and folder it needs:
   - This CLAUDE.md
   - Relevant source files it will read/modify
   - API contracts, data shapes, shared types
   - What the feature should do (requirements)
   - What it should NOT touch (constraints)
4. **Mark the session** in TODO.md or the Myrlin Workbook UI with:
   - What the session was created for (feature name)
   - When it was created (ISO timestamp)
   - Status (in-progress / completed / failed)
   - When it finished
5. **Feature session does NOT merge** — it commits to its branch and closes
6. **Orchestrator (main session) merges** — review the branch, resolve conflicts, merge to dev/main
7. **Clean up** — remove the worktree after merge: `git worktree remove ../cwm-feat-<feature-name>`

### Why

- Fresh context per feature = fewer hallucinations and drift
- Isolated branches = safe to experiment, easy to revert
- Orchestrator controls integration = no blind merges
- Audit trail = every feature has a clear lifecycle

### Task Tracking

**ALWAYS use the Todo tool proactively.** Every feature, bug fix, and task should be tracked:
- Create tasks at the start of work (TaskCreate)
- Mark in_progress when starting (TaskUpdate)
- Mark completed when done (TaskUpdate)
- Clean up stale tasks regularly (TaskList → TaskUpdate with deleted status)

## File Structure
```
claude-workspace-manager/
├── CLAUDE.md          # This file
├── src/               # Source code
├── test/              # Tests
├── screenshots/       # UI screenshots
├── state/             # Persisted workspace state (JSON)
└── dist/              # Built output (if applicable)
```

## Design System (GSD Agents: READ THIS)

This project uses the **Myrlin Workbook** design system. All UI/frontend work MUST follow:

1. **`C:\Users\Arthur\.claude\skills\ui-ux-pro-max.md`** - Full design skill with quality gates, MCP tools, and animation system
2. **`C:\Users\Arthur\.claude\UI_DESIGN_STRATEGY.md`** - Design philosophy and token system
3. **Catppuccin themes** (Mocha default, 4 flavors), CSS custom properties for all colors
4. **Typography**: Plus Jakarta Sans (sans), JetBrains Mono (mono)
5. **Animation**: 150-200ms transitions, skeleton loaders, no spinners
6. **North Star**: Linear, Raycast, Vercel, Arc, Warp

**Code standards**: `C:\Users\Arthur\.claude\skills\code-standards.md` - comments on every function, semantic versioning, changelog

**Available design MCPs**: stitch (mockups), nanobanana (image gen), visual-qa (screenshots)

## Mobile App Build - Orchestration Rules (NON-NEGOTIABLE)

> These rules govern the Myrlin Mobile build process. Every session involved in mobile development MUST follow these.

### Execution Model: Two-Tier Contract-Driven Orchestration

- **Orchestrator** (main session): Holds the master plan, all contracts, all type definitions. Creates plans, spawns agents, verifies integration. Does NOT write feature code directly.
- **Feature Agents** (subagents): Build individual features/screens to spec. Receive contracts + foundation code + their feature spec. Report back to orchestrator.
- **NO middle management layer.** Never spawn a "manager" agent that spawns its own subagents. Two tiers only: orchestrator -> worker agents.

### Agent Requirements (NON-NEGOTIABLE)

- **ALL spawned agents MUST use `model: "opus"`** with extended/deep thinking. No exceptions, no cost optimization, no "this is simple enough for sonnet." Opus only.
- ALL agent prompts MUST include: "Use your deepest reasoning for every decision. Think through edge cases, integration points, and failure modes before writing code."
- ALL agents receive the shared contracts file (types, API shapes, navigation routes, theme tokens) plus their specific task spec.
- ALL agents work in git worktrees for isolation.
- Agents MUST commit their work before completing. Orchestrator merges.

### Orchestrator Discipline

The orchestrator:
- **DOES:** Plan, define contracts, spawn agents, verify integration, merge branches, update plans, fix small integration glitches (< 20 lines)
- **DOES NOT:** Write feature screens, implement components, write styles, build full features directly
- **Exception:** Foundation phase may be built by orchestrator if the scope is small enough, OR by a single dedicated agent

### Phase Gates

1. Foundation phase MUST complete and be verified before any feature agents spawn
2. Feature agents within the same phase MAY run in parallel (up to 3 concurrent)
3. After each phase completes, orchestrator MUST verify integration before proceeding:
   - Build succeeds with zero errors
   - Navigation works between all screens
   - Shared state connects properly
   - Theme applies consistently
   - No TypeScript errors
4. If verification fails, orchestrator fixes integration issues or re-briefs affected agents

### Contract-First Development

- All TypeScript types, API interfaces, navigation routes, and component interfaces MUST be defined in the design doc BEFORE any agent writes code
- Agents build to these contracts. If a contract is wrong, the orchestrator updates it and re-briefs affected agents.
- Contracts live in shared files (`types/`, `api/`, `theme/`) that all agents import from
- Any agent that needs to deviate from a contract MUST document why in their commit message

### Mobile App Target: Full Parity

The mobile app MUST replicate every feature of the web GUI. No features are "desktop only" unless physically impossible on mobile (e.g., multi-pane terminal grid becomes swipeable single-pane). Every interaction, every screen, every setting. The mobile app is not a companion; it is a complete client.

---
> Source: [therealarthur/myrlin-workbook](https://github.com/therealarthur/myrlin-workbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
