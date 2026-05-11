## agent-knowledge

> **STOP. READ THIS FIRST.**

# THIS IS CLAUDE.md - YOUR MEMORY FILE

**STOP. READ THIS FIRST.**

This file is YOUR (Claude's) project memory. It is NOT user documentation. It is NOT a README.

| File | Purpose | Audience |
|------|---------|----------|
| **CLAUDE.md** (this file) | Claude Code's memory - scripts, workflows, coding rules | YOU (Claude) |
| **README.md** | User-facing documentation - features, installation, API | HUMANS (users) |

**When to update this file:** When scripts, CI/CD workflows, build processes, or coding conventions change.

**Keep this file LEAN.** This entire file loads into your context every session. Be concise. No prose. No redundancy. Every line must earn its place.

**CLAUDE.md is hierarchical.** Any subdirectory can have its own CLAUDE.md that auto-loads when you work in that directory. Use this pattern:
- **Root CLAUDE.md** (this file): Project-wide info - scripts, CI/CD, general conventions
- **Subdirectory CLAUDE.md**: Directory-specific context scoped to files below it
- Nest as deep as needed - each level inherits from parents

**Stay DRY with includes.** Use `@path/to/file` syntax to import content instead of duplicating. Not evaluated inside code blocks.

---

## Package Manager

**Use `bun`** - not npm or yarn. All scripts should be run with `bun run <script>`.

---

## Scripts

**Development:**
- `bun run build` - Compile TypeScript
- `bun run test:run` - Run tests once
- `bun run precommit` - Full validation (lint, typecheck, tests, build)
- `bun run prerelease` - Full quality checks (format, lint, deadcode, typecheck, coverage, build)

**Versioning (after code changes):**
- `bun run version:patch` - Runs `prerelease` checks, then bumps patch version
- `bun run version:minor` - Runs `prerelease` checks, then bumps minor version
- `bun run version:major` - Runs `prerelease` checks, then bumps major version

Note: Version scripts run full quality checks BEFORE bumping to prevent broken releases.

**Releasing (Fully Automated):**
1. Bump version: `bun run version:patch` (or minor/major)
2. Commit: `git commit -am "chore: bump version to X.Y.Z"`
3. Push: `git push`
4. **Done!** GitHub Actions handles the rest automatically

**What happens automatically:**
- `CI` workflow runs (lint, typecheck, tests, build)
- `Auto Release` workflow waits for CI, then creates & pushes tag
- `Release` workflow creates GitHub release
- `Update Marketplace` workflow updates `chris-xperimntl/xperimntl-marketplace`

## GitHub Actions Workflows

**CI Workflow** (`.github/workflows/ci.yml`)
- Triggers: Push to main, pull requests, tag pushes
- Runs: Lint, typecheck, tests, build
- Required to pass before auto-release creates tag

**Auto Release Workflow** (`.github/workflows/auto-release.yml`)
- Triggers: Push to main
- Waits for: CI workflow to pass
- Checks: If version in package.json has no matching tag
- Creates: Tag and pushes it (triggers Release workflow)
- Uses: `MARKETPLACE_PAT` for checkout/push (GITHUB_TOKEN can't trigger other workflows)
- **This is what makes releases fully automated**
- **Important:** Must use PAT, not GITHUB_TOKEN, to allow tag push to trigger Release workflow

**Release Workflow** (`.github/workflows/release.yml`)
- Triggers: Tag push (v*)
- Creates: GitHub release with auto-generated notes

**Update Marketplace Workflow** (`.github/workflows/update-marketplace.yml`)
- Triggers: After Release workflow completes (via `workflow_run`)
- Waits for: CI workflow success (via `wait-on-check-action`)
- Updates: `chris-xperimntl/xperimntl-marketplace` plugin version automatically
- Requires: `MARKETPLACE_PAT` secret (repo-scoped PAT with write access to marketplace repo)
- Note: Uses `workflow_run` trigger because GitHub prevents `GITHUB_TOKEN` workflows from triggering other workflows

## Distribution Requirements

**`dist/` MUST be committed to git** - This is intentional, not an oversight:

1. **Claude Code plugins are copied to a cache during installation** - no build step runs
   - Plugins need pre-built files ready to execute immediately
   - Source: https://code.claude.com/docs/en/discover-plugins

2. **npm publishing uses `files` array** - `package.json` `files` field controls what npm publishes AND what Claude Code caches. Must include all runtime files (dist, commands, skills, hooks, scripts, etc.)

**After any code change:**
1. Run `bun run build` (or `bun run precommit` which includes build)
2. Commit both source AND dist/ changes together

## Project Rules

Modular rules are in `.claude/rules/` and auto-load:

- `code-quality.md` - Fail early/fast, strict typing, no fallback code
- `git.md` - No --no-verify on commits
- `versioning.md` - Use version:* commands, push to main for releases

## Plugin Hooks

Hooks are defined in `hooks/hooks.json`. Key hooks:

| Hook | Event | Behavior |
|------|-------|----------|
| `auto-setup.sh` | SessionStart | Async - auto-installs deps if missing, re-runs on upgrade |
| `check-ready.sh` | SessionStart | Fast validation, non-blocking |
| `posttooluse-ak-reminder.py` | PostToolUse (Grep\|Read) | Reminds about BK after reading deps |
| `posttooluse-web-research.py` | PostToolUse (WebFetch) | Suggests indexing libs from source URLs |
| `posttooluse-websearch-ak.py` | PostToolUse (WebSearch) | Reminds to use BK for indexed libs |
| `job-status-hook.sh` | UserPromptSubmit | Async - surfaces background jobs |
| `userpromptsubmit-ak-nudge.py` | UserPromptSubmit | Injects BK reminder for dependency-related prompts |
| `stop-ak-check.py` | Stop | Blocks once if deps accessed but BK never consulted |

**Activation strategy:** Directive skill description (Variant C: "ALWAYS invoke...Do not X directly") + prescriptive MCP tool descriptions + context injection hooks. Based on 650-trial research showing directive descriptions achieve 100% activation vs 50% baseline.

**Async vs Sync Hooks:**
- **Async hooks** (`"blocking": false`) run in background. Output arrives next turn. Use for slow operations.
- **Sync hooks** (`"blocking": true`) block until complete. Default timeout: 60s. Use for validation/injection.

**Timeout/Failure Behavior:**
- Sync hook timeout → Claude continues without hook output
- Async hook failure → Error surfaces next turn, doesn't block session
- Hook exit code > 0 → Treated as failure, logged to `.agent-knowledge/logs/`

## Store Schema Versioning

| Version | Has `modelId` | Searchable | Incremental Index |
|---------|---------------|------------|-------------------|
| v1 (legacy) | No | No | No |
| v2 (current) | Yes | Yes | Yes |

**Migration path:** Run `bun run index <store> --force` → `upgradeStoreSchema()` adds `modelId` → store becomes searchable.

**Key invariant:** `modelId` only set after successful reindex. Before that, store has no embeddings.

**Implementation:**
- CLI upgrade: `src/cli/commands/index-cmd.ts:74`
- Background upgrade: `src/workers/background-worker.ts:228`
- MCP filters v1 stores gracefully: `src/mcp/handlers/search.handler.ts:66`

## MCP Debugging

**MCP server is FRAGILE.** When it fails, use `/agent-knowledge:doctor` first (works even when MCP is broken).

| Issue | Symptom | Fix |
|-------|---------|-----|
| Peer dependency conflict | `npm ERR! ERESOLVE` | `--legacy-peer-deps` in setup.sh and bootstrap.ts |
| Node.js v24 | Native module compilation fails | Use Node v20.x or v22.x LTS |
| Wrong cached version | Old bootstrap.ts runs, missing fixes | mcp-wrapper.sh uses `sort -V -r` for latest |
| Missing build tools | `make not found` | `build-essential` (Linux) or Xcode CLI (macOS) |

**Key files for MCP:**
- `scripts/mcp-wrapper.sh` - Finds plugin, runs bootstrap (version sorting critical!)
- `src/mcp/bootstrap.ts` - Installs deps, starts MCP server
- `scripts/setup.sh` - Manual/auto setup with npm install

**Logs:** `.agent-knowledge/logs/app.log`

---
> Source: [chris-xperimntl/agent-knowledge](https://github.com/chris-xperimntl/agent-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
