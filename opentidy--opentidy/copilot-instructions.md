## opentidy

> Open-source personal AI assistant. Manages administrative tasks via autonomous AI agent sessions (agent-agnostic: Claude Code, Gemini CLI, Copilot CLI), with hook-based guardrails and a web app. Distributed via Homebrew.

# CLAUDE.md

## Project: OpenTidy

Open-source personal AI assistant. Manages administrative tasks via autonomous AI agent sessions (agent-agnostic: Claude Code, Gemini CLI, Copilot CLI), with hook-based guardrails and a web app. Distributed via Homebrew.

**Repo:** `opentidy/opentidy` (GitHub org)
**Install:** `curl -fsSL https://opentidy.com/install.sh | bash`
**CLI:** `opentidy` (`opentidy setup`, `opentidy start`, `opentidy doctor`, etc.)
**License:** AGPL-3.0-only
**Specification:** `docs/specification.md`

## CRITICAL: Open-Source & Privacy

**This is a PUBLIC open-source repository. Every file, every commit, every diff is visible to everyone on the internet.**

### Zero personal information, no exceptions

- **NEVER** commit real names (except copyright holder in LICENSE/CLA), email addresses, phone numbers, physical addresses, account IDs, chat IDs, or any PII
- **NEVER** commit API keys, tokens, bearer tokens, passwords, or secrets of any kind
- **NEVER** commit URLs pointing to personal infrastructure (Cloudflare tunnels, self-hosted services, personal domains)
- **NEVER** reference specific user accounts, Telegram chat IDs, Gmail addresses, or real contacts in code, comments, configs, or fixtures
- **Fixtures and test data** must use only fictitious data: `example.com` emails, placeholder names ("Alice", "Bob"), fake IDs (`chat-123`), generic URLs
- **Config files** committed to the repo must contain only placeholders or empty values. Real config lives in `~/.config/opentidy/` (gitignored)

### Before every commit

Review the full diff. If **any** of the following appear, **stop and sanitize**:
- Real email addresses or phone numbers
- Telegram/WhatsApp chat IDs or bot tokens
- Bearer tokens, API keys, or OAuth credentials
- Personal domain names or tunnel URLs
- Real company names in fixtures (use generic alternatives)
- Paths containing usernames (use `~` or `$HOME` notation)

### License headers

Every source file (`.ts`, `.tsx`) must have this SPDX header as the first lines:

```typescript
// SPDX-License-Identifier: AGPL-3.0-only
// Copyright (c) 2026 Loaddr Ltd
```

## Commands

```bash
pnpm install                           # install all workspaces
pnpm build                             # build all packages
pnpm dev                               # dev mode (backend + web parallel)
pnpm test                              # vitest (backend)
pnpm test:e2e                          # playwright (web)
pnpm --filter @opentidy/backend test   # backend tests only
pnpm --filter @opentidy/web dev        # web dev only
pnpm --filter @opentidy/shared build   # shared types only
```

## Architecture

### 8 guiding principles

1. **Speed is not a criterion** (admin tasks, not real-time)
2. **AI agent CLI is the execution engine**: agent-agnostic (Claude Code, Gemini CLI, Copilot CLI), skills, MCP, browser, session resume
3. **Budget is not a constraint** (Claude Max, no token compromises)
4. **Intelligence lives in Claude, not the code**: backend does plumbing, Claude decides
5. **No interruption, isolated parallelism**: each task = its own session
6. **The assistant runs quietly in the background**: hybrid events + crons
7. **Quick/interactive actions = specialized tool**, not the core system
8. **Continuous improvement**: gaps.md = natural backlog

### Monorepo

```
opentidy/
├── pnpm-workspace.yaml
├── packages/
│   └── shared/              # TypeScript types, Zod schemas
├── apps/
│   ├── backend/             # Hono API, daemon, features (VSA)
│   └── web/                 # React SPA, Vite, features (VSA)
├── workspace/               # runtime: tasks, state.md, artifacts (gitignored)
└── docs/                    # public documentation
```

### Vertical Slice Architecture (VSA)

Backend and frontend follow VSA: code is organized by feature/domain, not by technical layer. Each feature directory is self-contained: route + handler + logic + colocated tests. An agent opens one directory and has full context.

**Backend** (`apps/backend/src/`):
```
features/
  tasks/           # CRUD, state.md parsing, title generation
  sessions/       # launch, stop, take-over, hand-back, post-session, crash recovery
  triage/         # webhook, mail/SMS readers, classify, route dispatch
  memory/         # CRUD, extraction agents, injection, lock
  suggestions/    # list, approve, dismiss, file parser
  ameliorations/  # list, resolve, ignore, gaps.md parser
  hooks/          # POST /api/hooks handler
  notifications/  # Telegram sender, notification store
  checkup/        # periodic sweep, workspace watcher
  terminal/       # ttyd bridge, port route
  system/         # health, reset, audit, processes, SSE events, test tasks
shared/           # cross-cutting: database, locks, dedup, spawn-agent, agents/, SSE, auth, config, paths
cli/              # CLI commands (setup/, doctor, status, logs, update, uninstall)
boot/             # periodic tasks setup (intervals, watchdog, crash recovery)
```

**Frontend** (`apps/web/src/`):
```
features/
  tasks/           # TaskCard, TaskDetail, Sidebar, StateRenderer
  sessions/       # SessionCard, SessionOutput
  ameliorations/  # AmeliorationCard, Ameliorations page
  memory/         # Memory page
  terminal/       # Terminal page, ProcessOutput, PlainTextOutput, LiveProcessOutput
  nouveau/        # Nouveau page
  home/           # Home page
shared/           # Layout, nav, ErrorBanner, InstructionBar, SuggestionCard, store, api, i18n, utils
```

**Cross-feature dependency rules** (unidirectional):
```
triage → sessions → tasks
checkup → sessions → tasks
```
All other features are standalone. Cross-feature deps are injected via `deps` parameter, not direct imports.

### Backend features detail

**Triage** (`features/triage/`):
- Webhooks (Gmail), watchers (SMS/WhatsApp), user instructions
- Dedup by content hash, classify via agent one-shot, dispatch results

**Sessions** (`features/sessions/`):
- Autonomous mode (default): spawns agent CLI as child process via `spawn-agent.ts`
- Interactive mode ("Take Over"): kill process → tmux → ttyd → "Hand Back"
- Post-session agent: memory extraction + gaps + journal verification
- Crash recovery at startup

**Tasks** (`features/tasks/`):
- Markdown files = task state (state.md, checkpoint.md, artifacts/)
- `## Waiting For` section = resume criteria
- Title generation via agent one-shot

**Hooks** (`features/hooks/`):
- `POST /api/hooks`: audit + SSE for lifecycle events

**Notifications** (`features/notifications/`):
- Telegram (grammy): checkpoint notifications, completions, escalations

**Shared** (`shared/`):
- SQLite (`better-sqlite3`), PID locks, dedup, agent spawner + semaphore, SSE emitter, auth middleware, config, paths
- Agent abstraction: `shared/agents/`: `AgentAdapter` interface, `resolveAgent()` registry, Claude adapter

### Agent Abstraction

OpenTidy is agent-agnostic. The `AgentAdapter` interface (`shared/agents/types.ts`) abstracts CLI differences:
- **Claude Code** (stable): `claude -p`, `--system-prompt`, `--resume`, `--allowedTools`, PreToolUse hooks
- **Gemini CLI** (experimental, not yet implemented): candidate
- **Copilot CLI** (experimental, not yet implemented): candidate

Resolution order: `OPENTIDY_AGENT` env → `--agent` flag → `config.json.agent` → fallback `claude`

Key files: `spawn-agent.ts` (spawner), `agents/claude.ts` (adapter), `agents/registry.ts` (resolver)

### Instruction files (2 levels of session context)

**Level 1**, `workspace/INSTRUCTIONS.md` (global, written once):
- Identity, style, security rules, expected formats, available tools
- Copied to agent-native file (`CLAUDE.md`, `GEMINI.md`, `AGENTS.md`) at spawn time

**Level 2**, `workspace/<task>/INSTRUCTIONS.md` (generated by backend at each launch):
- Task objective, confirm mode, event/instruction, contacts
- Copied to agent-native file at spawn time, stale files from other agents cleaned up

### Permission System

**Architecture:** module manifests declare `toolPermissions`, each tool tagged as `safe` or `critical`, scoped `per-call` or `per-task`. The user chooses a permission level per module: `allow` / `confirm` / `ask`.

**3 presets:**
- `supervised`: everything is `ask` (prompt user before each action)
- `autonomous`: critical tools are `confirm`, safe tools are `allow`
- `full-auto`: everything is `allow`

**At session launch**, `buildAllowedTools()` produces an `--allowedTools` list from the active manifests. A `type: "command"` PreToolUse hook intercepts `confirm`-level tool calls and calls `POST /api/permissions/check` before proceeding.

**Confirm flow:** backend sends a Telegram notification with an AI-generated summary of the action → waits for user approve/deny → deterministic outcome, no AI gatekeeping.

**AI role:** limited to generating human-readable summaries for notifications. All decision logic is deterministic.

**Lifecycle hooks:** `plugins/opentidy-hooks/guardrails.json`, `type: "command"` hooks only (Stop/SessionEnd audit, non-blocking). Lifecycle is managed by process exit.

### Data flow

```
Gmail webhook / SMS watcher / WhatsApp watcher / Telegram / Web app
    → Receiver (dedup + agent triage)
    → Launcher (agent child process, autonomous mode)
    → Agent works (--allowedTools enforces permissions, PreToolUse hook intercepts confirm-level tools)
    → state.md updated / checkpoint.md if blocked / ## Waiting For if waiting for external
    → Process exit → handleAutonomousExit() → cleanup lock, notification, post-session agent (memory)
    → Optional: "Take Over" → kill process → interactive tmux → "Hand Back" → autonomous relaunch
    → Periodic sweep → check tasks, launch sessions
```

### Agent sessions

Sessions are spawned via `spawn-agent.ts` which delegates to the active `AgentAdapter`. The adapter builds CLI args for its agent.

**Autonomous mode (default)**: Node.js child process via `adapter.buildArgs({ mode: 'one-shot', ... })`:
```bash
# Example with Claude adapter:
claude -p --strict-mcp-config --mcp-config '{}' --system-prompt "..." "<instruction>"
# Process runs in the task cwd, exit = end of session
```

**Interactive mode ("Take Over")**: tmux via `adapter.buildArgs({ mode: 'interactive', ... })`:
```bash
tmux new-session -d -s opentidy-<task-id> \
  "cd workspace/<task-id> && <agent-binary> <adapter-generated-args>"
```

**One-shot calls** (triage, sweep, memory): all use `adapter.buildArgs({ mode: 'one-shot', ... })` then `spawnAgent()`.

**Tool permissions**: `--allowedTools` built from module manifests at launch. Confirm-level tools intercepted by PreToolUse hook → `POST /api/permissions/check` → user approve/deny via Telegram.

### Browser: Camoufox

Each session has its own Camoufox instance with an isolated profile. Not Chrome/Playwright.
- Full parallelism (no browser lock)
- Anti-detection
- Persistent sessions per profile (cookies, logins preserved)

## Frontend (Web app)

**Tech:** React 19, Vite, React Router v7, Tailwind CSS v4 (CSS-first), Zustand, xterm.js

**Pages:**
- Home: active tasks + suggestions + gaps
- Task detail: state.md, checkpoint, terminal (xterm.js → ttyd/tmux, interactive mode only), artifacts
- New task: instruction + confirm mode
- Notifications: history

**SSE**: native EventSource → Zustand store in real-time

**React 19**: NEVER use `useMemo`, `useCallback`, or `React.memo`; React Compiler handles memoization.

## Code Style

- **TypeScript strict** everywhere
- **Zod** for validation (schemas in `packages/shared`)
- **Factory functions**: no classes. Each module exports `createX()` returning an interface. Enables easy mocking in tests.
- **SSOT**: never duplicate types, constants, or state
- **Progressive Logging**: `console.error`/`console.warn` with context. `console.log` at boundaries (API route entry, hook handler, Claude spawn). Prefix `[service]` (e.g., `[launcher]`, `[receiver]`, `[triage]`). No logs in loops.
- **Agent Timeouts**: All agent one-shot calls (triage, checkup, title gen, memory agents) must have a timeout of **1h minimum** (`3_600_000`). Agents can be slow under load (rate limits, queue). The timeout is a zombie guard, not a perf constraint. NEVER set a short timeout (30s, 60s) on an agent call.

## Testing

**Vitest** for backend, **Playwright** for frontend E2E.

- Tests are **colocated** with source files (`create.ts` + `create.test.ts` in same directory)
- Factory function mocking (no DI framework)
- DB tests: workspace files + SQLite (planned) in tmpdir
- **Every code change must include appropriate tests**

## Test Tasks Safety

When generating test tasks for OpenTidy (test tasks, debug, feature validation):
- **NEVER irreversible actions**: no sending real emails to third parties, no transactions, no public posts
- **Emails only to the configured user**, never to real contacts
- **Fictitious destinations**: use `example.com` for fake email addresses
- **Safe navigation**: public sites only (Wikipedia, CoinGecko, Booking), no login on sensitive accounts

## Git

### Conventional commits (required; Release Please generates changelogs from them)

Format: `type(scope): message`

| Type | When | Version bump |
|------|------|-------------|
| `feat` | New feature or capability | patch (pre-1.0), minor (post-1.0) |
| `fix` | Bug fix | patch |
| `refactor` | Code change that neither fixes a bug nor adds a feature | none |
| `test` | Adding or updating tests | none |
| `docs` | Documentation only | none |
| `chore` | Maintenance, deps, CI config | none |
| `ci` | CI/CD changes | none |

Scopes: `backend`, `web`, `shared`, `cli`, `hooks`, `docs`

Breaking changes: add `!` after scope → `feat(backend)!: redesign triage pipeline` or add `BREAKING CHANGE:` in commit body.

### Commit frequently

**Commit after every logical unit of work.** Don't accumulate changes. Examples:
- Added a new function + its tests → commit
- Fixed a bug → commit
- Refactored a module → commit
- Updated config or docs → commit

Small, focused commits make PRs easy to review and give Release Please good material for changelogs.

### Workflow

**Repo admins** can push directly to `main` for quick fixes.

**Everyone else (agents, contributors):**
1. Create a feature branch from `main`: `feat/short-description`, `fix/short-description`, `refactor/short-description`
2. Commit frequently with conventional commits
3. Push the branch and open a PR to `main`
4. PRs must pass all tests, follow conventional commits, contain zero secrets/PII
5. Never push directly to `main`

**Agents (Claude Code sessions):**
- Always work on a feature branch, never on `main`
- Commit as you go, after each logical step, not at the end
- Push and open a PR when done (use `gh pr create`)
- Never add `Co-Authored-By`
- Never force-push or rewrite history

### Contributor specifics

- External contributors: fork → feature branch → PR to `main`
- CLA signature required on first PR (CLA Assistant bot)
- PR template: `.github/PULL_REQUEST_TEMPLATE.md`
- Contributing guide: `docs/contributing.md`

## Distribution & Deployment

**Installation:**
```bash
curl -fsSL https://opentidy.com/install.sh | bash
# Handles: Homebrew, Node.js 22, opentidy formula, setup wizard, background service
```

**Uninstall:**
```bash
curl -fsSL https://opentidy.com/install.sh | bash -s -- --uninstall
# or
opentidy uninstall
```

**Service management (Homebrew LaunchAgent):**
```bash
brew services start opentidy    # start background service
brew services stop opentidy     # stop background service
brew services restart opentidy  # restart background service
opentidy start                  # alternative: start via CLI
opentidy stop                   # alternative: stop via CLI
```

**CLI commands:**
```bash
opentidy start       # start the backend server
opentidy stop        # stop the backend server
opentidy restart     # restart the backend server
opentidy setup       # interactive setup wizard (modular, rerunnable)
opentidy doctor      # verify deps, config, permissions, connectivity
opentidy status      # service state, version, uptime
opentidy update      # check and apply updates (runs brew upgrade opentidy)
opentidy logs        # tail log files (also at $(brew --prefix)/var/log/opentidy.log)
opentidy uninstall   # scoped removal (service, config, data, tunnel)
```

**CI/CD:** GitHub Actions (`release.yml`): tag `v*` → test → build → tarball → GitHub Release → update Homebrew formula

**Config:** `~/.config/opentidy/config.json` (secrets, bearer token, port, update prefs, user info, MCP service states)

**Dashboard:** `http://localhost:5175`

**Agent isolation:** each agent gets its own config directory under `~/.config/opentidy/agents/<name>/` (e.g., `CLAUDE_CONFIG_DIR=~/.config/opentidy/agents/claude/`). Config generated by `adapter.writeConfig()` during `opentidy setup`: hooks, permissions, MCP servers. User's personal agent config is never touched.

**Remote access:** Cloudflare Tunnel (Zero Trust) + bearer token auth

**Auto-update:** `opentidy update` checks GitHub Releases and runs `brew upgrade opentidy` with health check and automatic rollback on failure.

## Secrets & Auth

- **Agent auth**: each agent uses its own auth mechanism (Claude = OAuth/Max subscription, Gemini = Google account, Copilot = GitHub subscription). Never hardcode API keys.
- **Config**: `~/.config/opentidy/config.json` (Telegram tokens, bearer token, port, user info, MCP states), never committed
- **Cloudflare**: credentials managed by `cloudflared` itself
- **Never hardcode secrets** in code, committed .env files, or docker-compose
- **Public repo rule**: if a value is secret or personal, it goes in runtime config (`~/.config/opentidy/`), never in the repo. Code must read from config at runtime, not from hardcoded values.

## Key Paths

- `apps/backend/src/features/`: backend feature slices (VSA)
- `apps/backend/src/shared/`: backend cross-cutting code (db, locks, auth, config, sse)
- `apps/backend/src/cli/`: CLI commands (setup, doctor, status, update, logs, uninstall)
- `apps/backend/src/server.ts`: route assembler (imports from features/, mounts on Hono)
- `apps/backend/src/index.ts`: boot orchestrator (deps → server → periodic → listen)
- `apps/web/src/features/`: frontend feature slices (VSA)
- `apps/web/src/shared/`: frontend shared (Layout, store, api, i18n, utils)
- `packages/shared/src/`: shared types, Zod schemas
- `bin/opentidy`: CLI entry point (installed via Homebrew)
- `apps/backend/config/claude/`: Claude Code config template
- `plugins/opentidy-hooks/`: PreToolUse hooks (lifecycle audit)
- `workspace/`: runtime data (tasks, state.md, artifacts), **gitignored**
- `workspace/INSTRUCTIONS.md`: global level 1 prompt (source of truth, not gitignored)
- `apps/backend/src/shared/agents/`: agent adapters (claude.ts, registry.ts)
- `apps/backend/src/shared/spawn-agent.ts`: agent-agnostic process spawner
- `apps/backend/src/features/modules/`: module registry, manifest loader, tool permission resolver
- `apps/backend/src/features/permissions/`: confirm flow, `POST /api/permissions/check`, pending store
- `plugins/opentidy-hooks/guardrails.json`: lifecycle hook rules (Stop/SessionEnd audit)
- `/tmp/opentidy-locks/`: PID lock files runtime
- `docs/`: public documentation
- `.github/workflows/release.yml`: CI/CD pipeline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/opentidy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
