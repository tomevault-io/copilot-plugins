## koda

> This file is for AI coding assistants (Cursor, Claude Code, Copilot, and

# AGENTS.md — Rules for AI coding assistants working on Koda

This file is for AI coding assistants (Cursor, Claude Code, Copilot, and
similar tools) editing the Koda codebase. It captures architectural
decisions, conventions, and operational rules that would otherwise require
re-learning on every session.

**This file is NOT for the running Koda daemon itself** — that's what
`~/.koda/soul.md` is for. Koda's runtime personality lives in user state;
this file lives in the code repo and describes how to write/modify the
code.

## What Koda is

Koda is a TypeScript daemon built on Anthropic's Claude Agent SDK. It runs
as a persistent background process (via pm2) that handles autonomous
content production for one solo operator. The operator controls it through
Discord; Koda executes tasks via scheduled cron jobs, webhooks, and MCP
servers.

Koda is **explicitly single-tenant**. It is not a multi-user product, not a
SaaS, and should not be refactored toward either without a deliberate
decision. The architecture assumes one human, one `~/.koda/` directory,
one set of OAuth tokens.

## Architecture (read before editing anything non-trivial)

### Entry points

- `src/index.ts` — process entry point (daemon mode). Wires the agent, Discord
  bot, scheduler, webhook server, voice support, and teleport.
- `src/koda.ts` — CLI binary entry point (`#!/usr/bin/env node` shebang).
  Subcommand router. Lives at `dist/koda.js` after build, symlinked globally
  via `npm link`.
- `src/cli.ts` — legacy interactive REPL. Untouched. `npm run cli` still
  works for anyone who wants it. Don't delete without replacing the npm
  script.

### Core modules

- `src/agent.ts` — the persistent streaming `query()` session + isolated
  task runner (`runIsolatedTask`). The persistent session handles Discord
  messages and teleport context. Isolated tasks run with fresh context for
  scheduled jobs and ticks so they don't bloat the main session.
- `src/bot.ts` — Discord bot. Message routing, rate limiting, approval
  reactions, startup message (via `DISCORD_PROACTIVE_CHANNEL`), attachment
  handling, long-message chunking.
- `src/scheduler.ts` — cron-based task runner, tick loop (Sonnet isolated
  sessions), daily digest, auto-backup, missed-task recovery, self-healing.
- `src/dream.ts` — nightly memory consolidation cycle (03:07 daily, LLM-driven).
- `src/runtime.ts` — shared runtime helpers (content-type classification,
  memory freshness checks, observation writes).
- `src/patterns.ts` — reusable behavioral patterns: circuit breaker, state
  file queue, session registry.
- `src/skills.ts` — loads skills from `~/.koda/skills/` into the system prompt.
  Supports both flat `.md` files and directory-based `SKILL.md` format
  (AgentSkills / ClawHub compatible).

### Commands (CLI)

Each CLI subcommand lives in `src/commands/<name>.ts`, exports:
- `description: string`
- `runX(args: string[]): Promise<number>` (exit code)

Commands are registered in `src/koda.ts` → `COMMANDS` map. To add a new
command: create the file, export the two things, add a line to the map.
That's it.

Current commands: `init`, `status`, `update`, `logs`, `restart`, `doctor`,
`skills`, `health`.

### Tools (MCP)

Koda exposes custom tools to its own agent sessions via `src/tools/agent-tools.ts`.
These are Agent SDK MCP tools (NOT external MCP servers). They show up
in the agent's tool list as `mcp__agent-tools__<toolName>`.

External MCP servers (x-mcp, bluesky-mcp, youtube, gmail, etc.) are
loaded from `~/.koda/mcp-servers.json` at runtime via `getMcpServers()`
in `src/agent.ts`.

### Memory pipeline (Layer numbers)

Koda implements a 6-layer memory consolidation pipeline. **Do not break
the layer boundaries.** The design is documented in full at
`docs/memory-architecture.md`. Short version:

| Layer | Location | Producer | Consumer | Cadence |
|---|---|---|---|---|
| 1 — Bootstrap | `~/.koda/soul.md`, `user.md`, `goals.md`, `skills/` | User | Every agent session | Static |
| 2 — Observations | `~/.koda/data/observations.md` | Running agent (via `observe` tool) | Dream cycle | Continuous |
| 3 — Dream cycle | `src/dream.ts` | `dream_cycle` scheduled task | Layers 4 & 5 | 03:07 daily |
| 4 — Daily logs | `~/.koda/data/daily-logs/{date}.md` | `daily_digest` scheduled task | `learnings_review` | 21:00 daily |
| 5 — Learnings | `~/.koda/learnings.md` | `learnings_review` scheduled task | Every agent session | 07:45 daily |
| 6 — Search | `~/.koda/scripts/search-memory.sh` | User or agent | — | On demand |

Each layer has exactly one producer. If you're tempted to add a second
writer to any layer, stop and ask whether you're breaking the invariant.

## Authentication

Koda uses the Claude Agent SDK, which authenticates via one of two paths:

1. **Max subscription** (default): reads the Claude Code CLI's login state
   at `~/.claude/`. User runs `claude login` once, then Koda works
   transparently. No `ANTHROPIC_API_KEY` needed.

2. **API key**: if `ANTHROPIC_API_KEY` is set in `~/.koda/.env`, the SDK
   uses it instead. Useful for non-interactive servers or when Max quota
   runs out.

Koda is the "Agent SDK" class of runtime, which Anthropic has explicitly
confirmed remains eligible for Max subscription usage (confirmed by
Anthropic's Thariq Shihipar in The New Stack, April 2026). Do not
refactor Koda to wrap the Claude Code CLI instead — that would put it
in the "third-party harness" category which is banned from Max.

## Do not touch

- `dist/` — compiled output. Regenerated on every `npm run build`. Never
  edit directly. Never commit changes to dist (it's in `.gitignore`...
  or should be).
- `node_modules/` — obvious.
- `.env` at any level — never read, never log, never commit.
- `~/.koda/` when editing code — that's user state, separate from the
  repo. Changes to `~/.koda/` are the operator's business, not the
  codebase's.
- `ecosystem.config.cjs` env section — let `dotenv` own environment
  variables. Past versions of this file hardcoded `TICK_INTERVAL_MS=0`
  and caused silent failures on cold start. Never add env vars here.
- `~/.claude/` — Claude Code's own state directory. Koda reads from it
  (via `settingSources: ["user", "project"]` for plugin/skill discovery)
  but should never write to it.

## Conventions

### TypeScript

- Strict mode. Use `unknown` + narrowing, not `any`.
- ESM imports with `.js` extensions even when importing `.ts` files
  (this is the `"type": "module"` pattern).
- Prefer `node:` prefix for stdlib imports (`import { resolve } from "node:path"`).
- Async functions return `Promise<number>` for command exit codes, `Promise<void>`
  for fire-and-forget.

### Code style

- **No abstractions for single-use code.** Three similar lines is better
  than a premature helper function. Only extract when the same pattern
  appears in 3+ places.
- **Comments only where the logic isn't self-evident.** Don't narrate what
  the code obviously does. Do explain *why* when the reason is non-obvious
  (e.g., "ClawHub embeds metadata as a JSON blob on one line, not YAML").
- **No error handling for impossible cases.** Don't wrap every call in
  try/catch "just in case." Only catch at boundaries (user input, external
  APIs, filesystem operations on user-provided paths).
- **Fail loudly on unexpected state.** Silent fallbacks hide bugs. When
  something is wrong, console.error with context and exit with non-zero.

### CLI output

- ANSI colors when `process.stdout.isTTY`, plain text otherwise.
- Colors: `\x1b[32m` (green ✓ success), `\x1b[33m` (yellow ⚠ warning),
  `\x1b[31m` (red ✗ error), `\x1b[1m` (bold for headers).
- Use sparingly. Most output should be plain. Reserve color for status
  indicators and section headers.

### Git

- No `chore:` commit prefix. Use `feat:`, `fix:`, `docs:`, `refactor:`,
  `build:`, `ci:`.
- No `Co-Authored-By:` lines in commits.
- Commit messages should explain *why*, not *what*. The diff shows what.
- Never push without the operator's explicit approval. Never force-push
  to `main`. Never skip hooks (`--no-verify`) unless the operator asked.
- Never run destructive git operations (`reset --hard`, `clean -fd`, etc.)
  without explicit confirmation.

### Builds

- `npm run build` runs `tsc && chmod +x dist/koda.js`. Must pass cleanly
  before commit.
- `npm run dev` runs `tsx src/index.ts` (daemon in foreground).
- `npm run koda` runs `tsx src/koda.ts` (CLI in dev mode).
- Shebang `#!/usr/bin/env node` at the top of `src/koda.ts` — tsc preserves
  it through build. Don't remove.

## Before committing

Run locally:

1. `npm run build` — must pass
2. `koda doctor` — must report healthy (or only have warnings, no errors)
3. Review `git diff` yourself — don't trust an AI's judgment blindly on
   what changed
4. Stage specific files, not `git add -A` (avoids accidentally committing
   secrets or build artifacts)

Do NOT:

- Push to `main` without the operator saying "push"
- Add anything that requires `sudo` to install
- Introduce new runtime dependencies without asking (check `package.json`
  — if the diff adds a line under `dependencies`, flag it)
- Modify `~/.koda/` during code edits (wrong layer — that's user state)
- Add `ANTHROPIC_API_KEY` to `.env.example` as a required field (it's
  optional — Max sub is the default path)

## Common tasks

### Add a new CLI subcommand

1. Create `src/commands/<name>.ts` with `description` and `runX(args)`
2. Import and register in `src/koda.ts` → `COMMANDS` map
3. `npm run build` to verify it compiles
4. Test: `koda <name>` (via `npm link` — which you already did once)

### Add a new MCP tool

1. Add the tool definition in `src/tools/agent-tools.ts` using the `tool()`
   helper from `@anthropic-ai/claude-agent-sdk`
2. Add it to the `tools` array at the bottom (`createSdkMcpServer`)
3. If the tool is HIGH risk (modifies external state like posting to
   social, writing files, running shell), add its full name
   (`mcp__agent-tools__<toolName>`) to `HIGH_RISK_TOOLS` in `src/agent.ts`
4. Rebuild, restart the daemon with `pm2 restart koda --update-env`
5. Test via teleport or Discord

### Add a new scheduled task

1. Edit `~/.koda/tasks.json` (user state, not in the repo) — add an entry
   with `prompt`, `cron`, `type` (`silent` or `approval`), and optional
   `limits` (`maxTurns`, `maxBudgetUsd`)
2. The hot-reload file watcher picks up the change automatically — no
   restart needed
3. For templates that ship with the repo, edit `templates/tasks.example.json`
   instead

### Add a new skill

1. Create a markdown file in `~/.koda/skills/<name>.md` (flat format) OR
   `~/.koda/skills/<name>/SKILL.md` (directory format, ClawHub-compatible)
2. YAML frontmatter with `name` and `description` (required), `when`
   (optional — Koda-specific trigger hint)
3. Markdown body = the actual instructions
4. Restart daemon (`pm2 restart koda --update-env`) — skills load at
   startup, not hot-reloaded

### Fix a schema drift error from `koda doctor`

1. Check what doctor reported (missing env var, missing config field, etc.)
2. Update `~/.koda/.env` or `~/.koda/config.json` with the missing value
3. Re-run `koda doctor` to verify
4. If the drift is in `templates/.example.*` files (because you added a
   new required field in code), update the template too

### Restart the daemon safely

- From terminal: `koda restart "<reason>"` — reason is persisted and
  surfaced in the next Discord startup message
- From Discord: `@Koda restart yourself, reason: ...` — Koda calls the
  `restart_self` tool which writes the reason and calls pm2
- Never `pm2 restart koda` directly without a reason — the startup
  message will be blank and the operator won't know why

## Reference docs

- `docs/memory-architecture.md` — full 6-layer memory pipeline design,
  producers/consumers, cadence, invariants
- `README.md` — user-facing install and operational docs
- `install.sh` — the curl-bash installer (auto-installs prereqs on
  macOS/Linux, clones, builds, links, runs doctor, restarts daemon)
- `package.json` → `bin.koda` — CLI binary registration

## The test for "should I add this?"

When you're about to add a feature, abstraction, or file, ask:

1. **Does this solve a problem the operator has right now?** If no, don't
   build it. Don't build for hypothetical future use cases.
2. **Is there already a simpler way that uses existing patterns?** If yes,
   use that.
3. **Will this still be correct in 3 months without maintenance?** If no,
   it's probably premature.
4. **Can the operator understand this without reading a separate doc?**
   If no, either simplify or explain inline.

Koda is one operator's daemon. The right level of complexity is "the
minimum that makes today's task work and doesn't create tomorrow's
cleanup." Resist the urge to build frameworks.

## What NOT to refactor toward

Periodically, an AI assistant will suggest:

- Making Koda multi-tenant / multi-user
- Wrapping the Agent SDK in an abstraction layer "to support other providers"
- Replacing pm2 with a custom daemon manager
- Adding a web UI
- Building a plugin marketplace
- Splitting the repo into multiple packages

**All of these are wrong for Koda's scope.** If the operator ever decides
Koda should become a product, that's a separate project with a separate
repo. Don't preemptively refactor this one. Koda is solo-daemon-shaped
on purpose.

---
> Source: [kfuras/koda](https://github.com/kfuras/koda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
