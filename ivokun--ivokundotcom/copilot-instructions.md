## ivokundotcom

> > **Context:** This is a monorepo built with Bun, Astro, Hono, and Effect TS.

# AGENTS.md - Guidance for Coding Agents

> **Context:** This is a monorepo built with Bun, Astro, Hono, and Effect TS.
> **Primary Goal:** Maintain high code quality, strict typing, and deployment stability.

## 1. Build, Lint & Test Commands

### Root / Global
- **Type Check:** `bun run typecheck` (Checks all packages)
- **Database Migrations (Root):** `bun run db:up` (using dbmate)
- **Docker:** `bun run docker:up` / `bun run docker:down`

### Web (Astro + React)
- **Dev Server:** `bun --filter web dev`
- **Build:** `bun --filter web build`
- **Check:** `bun --filter web astro check`

### CMS (Effect TS + Kysely + SolidJS)
- **Dev Server:** `bun --filter '@ivokundotcom/cms' dev` (Runs server + SPA)
- **Dev Server Only:** `bun --filter '@ivokundotcom/cms' dev:server`
- **Dev SPA Only:** `bun --filter '@ivokundotcom/cms' dev:spa`
- **Build Binary:** `bun --filter '@ivokundotcom/cms' build`
- **Build SPA:** `bun --filter '@ivokundotcom/cms' build:spa`
- **Type Check:** `bun --filter '@ivokundotcom/cms' typecheck`
- **Lint:** `bun --filter '@ivokundotcom/cms' lint`
- **Format:** `bun --filter '@ivokundotcom/cms' format`
- **Test All:** `bun --filter '@ivokundotcom/cms' test`
- **Test Single File:** `cd cms && bun test src/services/post.service.test.ts`
- **Test Watch:** `cd cms && bun test --watch`
- **Migrations:** `bun --filter '@ivokundotcom/cms' db:up`
- **Seed Admin:** `bun --filter '@ivokundotcom/cms' seed:admin`

## 2. Code Style & Conventions

### General Guidelines
- **Package Manager:** Use `bun` (v1.1.38+) exclusively. NEVER use npm/yarn/pnpm.
- **Node Engine:** Requires Node.js >=22.17.0.
- **Lockfile:** Always commit `bun.lock`.
- **Secrets:** Never commit `.env` files.

### Formatting (Prettier)
- **Single Quotes:** Always use single quotes.
- **Trailing Commas:** Use `es5` style.
- **Print Width:** 100 characters.
- **Tab Width:** 2 spaces (never tabs).
- **Arrow Parens:** Always include (e.g., `(x) => x`).
- **End of Line:** LF (`\n`).

### Linting
- **Import Sorting:** Imports MUST be sorted using `simple-import-sort` plugin.
- **Parser:** Uses `@typescript-eslint/parser` for TS/TSX.
- **Astro:** Uses `astro-eslint-parser` for `.astro` files.

### TypeScript Rules
- **Strict Mode:** Enabled (extends `@tsconfig/node22`).
- **No Emit:** `tsc --noEmit` is used for checking; Bun/Esbuild/Vite handles emission.
- **Types:**
  - Use `interface` for object definitions (extensible).
  - Use `type` for unions/intersections.
  - Avoid `any`. Use `unknown` or distinct types.
- **Path Aliases:**
  - CMS: `~/*` maps to `./src/*`
  - Web: `@components/*`, `@layouts/*`, `@api/*`, `@utils/*`

### Effect TS Patterns (CMS)
- **Error Handling:** Avoid `try/catch`. Use `Effect.try`, `Effect.fail`, and `Effect.catchTag`.
- **Services:** Define services using `Context.Tag` (e.g., `class PostService extends Context.Tag('PostService')<...>()`).
- **Schemas:** Use `@effect/schema` for runtime validation (API requests/responses, DB models).
- **Pipelines:** Use `pipe()` for composition.
- **Error Types:** Define errors using `Data.TaggedError()` (see `src/errors.ts`).
- **Layers:** Use `Layer` for service composition and dependency injection.

### Naming Conventions
- **Variables/Functions:** `camelCase` (e.g., `getUser`, `isValid`).
- **Components/Classes:** `PascalCase` (e.g., `UserProfile`, `AuthService`).
- **Files:** `kebab-case` (e.g., `auth-service.ts`, `user-profile.tsx`).
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `MAX_RETRIES`).
- **Effect Tags:** Use PascalCase matching the service name.

### Project Structure
- **`web/`**: Astro frontend with React islands.
- **`cms/`**: Standalone CMS binary using Kysely (Postgres) and SolidJS admin panel.
- **`infra/`**: Terraform infrastructure definitions.
- **`docs/adr/`**: Architecture Decision Records.

### Workspace Configuration
Each package has its own `bunfig.toml` for proper module resolution:

| Package | Name | Filter |
|---------|------|--------|
| web | `web` | `bun --filter web` |
| cms | `@ivokundotcom/cms` | `bun --filter '@ivokundotcom/cms'` |

See [ADR-006](./docs/adr/006-bun-workspace-configuration.md) for details.

## 3. Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1.  **File Issues:** Create issues for any incomplete work or tech debt introduced.
2.  **Quality Gates:**
    - Run `bun run typecheck` at root.
    - Run tests for modified packages (e.g., `bun --filter '@ivokundotcom/cms' test`).
    - Run lint checks (e.g., `bun --filter '@ivokundotcom/cms' lint`).
3.  **Commit & Push:**
    - **Rebase First:** `git pull --rebase`
    - **Push:** `git push`
    - **Verify:** `git status` must show "up to date with origin".
4.  **Clean Up:** Remove temporary files, stashes, or dead branches.

**CRITICAL RULES:**
- **Never** stop before pushing. Local work is lost work.
- **Never** say "ready to push" without doing it. YOU are the agent; YOU push.
- **Retry** if push fails (network, conflicts). Resolve it.

## 4. Dependencies & Security

- **Secrets:** Never commit `.env` files.
- **Dependencies:** Install new packages with `bun add` (or `bun add -d` for dev).
- **Lockfile:** Always commit `bun.lock`.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **ivokundotcom** (712 symbols, 1695 relationships, 26 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/ivokundotcom/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |
| `detect_changes` | Pre-commit scope check | `gitnexus_detect_changes({scope: "staged"})` |
| `rename` | Safe multi-file rename | `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` |
| `cypher` | Custom graph queries | `gitnexus_cypher({query: "MATCH ..."})` |

## Impact Risk Levels

| Depth | Meaning | Action |
|-------|---------|--------|
| d=1 | WILL BREAK — direct callers/importers | MUST update these |
| d=2 | LIKELY AFFECTED — indirect deps | Should test |
| d=3 | MAY NEED TESTING — transitive | Test if critical path |

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/ivokundotcom/context` | Codebase overview, check index freshness |
| `gitnexus://repo/ivokundotcom/clusters` | All functional areas |
| `gitnexus://repo/ivokundotcom/processes` | All execution flows |
| `gitnexus://repo/ivokundotcom/process/{name}` | Step-by-step execution trace |

## Self-Check Before Finishing

Before completing any code modification task, verify:
1. `gitnexus_impact` was run for all modified symbols
2. No HIGH/CRITICAL risk warnings were ignored
3. `gitnexus_detect_changes()` confirms changes match expected scope
4. All d=1 (WILL BREAK) dependents were updated

## Keeping the Index Fresh

After committing code changes, the GitNexus index becomes stale. Re-run analyze to update it:

```bash
npx gitnexus analyze
```

If the index previously included embeddings, preserve them by adding `--embeddings`:

```bash
npx gitnexus analyze --embeddings
```

To check whether embeddings exist, inspect `.gitnexus/meta.json` — the `stats.embeddings` field shows the count (0 means no embeddings). **Running analyze without `--embeddings` will delete any previously generated embeddings.**

> Claude Code users: A PostToolUse hook handles this automatically after `git commit` and `git merge`.

## CLI

- Re-index: `npx gitnexus analyze`
- Check freshness: `npx gitnexus status`
- Generate docs: `npx gitnexus wiki`

<!-- gitnexus:end -->

<!-- rtk-instructions v2 -->
# RTK (Rust Token Killer) - Token-Optimized Commands

## Golden Rule

**Always prefix commands with `rtk`**. If RTK has a dedicated filter, it uses it. If not, it passes through unchanged. This means RTK is always safe to use.

**Important**: Even in command chains with `&&`, use `rtk`:
```bash
# ❌ Wrong
git add . && git commit -m "msg" && git push

# ✅ Correct
rtk git add . && rtk git commit -m "msg" && rtk git push
```

## RTK Commands by Workflow

### Build & Compile (80-90% savings)
```bash
rtk cargo build         # Cargo build output
rtk cargo check         # Cargo check output
rtk cargo clippy        # Clippy warnings grouped by file (80%)
rtk tsc                 # TypeScript errors grouped by file/code (83%)
rtk lint                # ESLint/Biome violations grouped (84%)
rtk prettier --check    # Files needing format only (70%)
rtk next build          # Next.js build with route metrics (87%)
```

### Test (90-99% savings)
```bash
rtk cargo test          # Cargo test failures only (90%)
rtk vitest run          # Vitest failures only (99.5%)
rtk playwright test     # Playwright failures only (94%)
rtk test <cmd>          # Generic test wrapper - failures only
```

### Git (59-80% savings)
```bash
rtk git status          # Compact status
rtk git log             # Compact log (works with all git flags)
rtk git diff            # Compact diff (80%)
rtk git show            # Compact show (80%)
rtk git add             # Ultra-compact confirmations (59%)
rtk git commit          # Ultra-compact confirmations (59%)
rtk git push            # Ultra-compact confirmations
rtk git pull            # Ultra-compact confirmations
rtk git branch          # Compact branch list
rtk git fetch           # Compact fetch
rtk git stash           # Compact stash
rtk git worktree        # Compact worktree
```

Note: Git passthrough works for ALL subcommands, even those not explicitly listed.

### GitHub (26-87% savings)
```bash
rtk gh pr view <num>    # Compact PR view (87%)
rtk gh pr checks        # Compact PR checks (79%)
rtk gh run list         # Compact workflow runs (82%)
rtk gh issue list       # Compact issue list (80%)
rtk gh api              # Compact API responses (26%)
```

### JavaScript/TypeScript Tooling (70-90% savings)
```bash
rtk pnpm list           # Compact dependency tree (70%)
rtk pnpm outdated       # Compact outdated packages (80%)
rtk pnpm install        # Compact install output (90%)
rtk npm run <script>    # Compact npm script output
rtk npx <cmd>           # Compact npx command output
rtk prisma              # Prisma without ASCII art (88%)
```

### Files & Search (60-75% savings)
```bash
rtk ls <path>           # Tree format, compact (65%)
rtk read <file>         # Code reading with filtering (60%)
rtk grep <pattern>      # Search grouped by file (75%)
rtk find <pattern>      # Find grouped by directory (70%)
```

### Analysis & Debug (70-90% savings)
```bash
rtk err <cmd>           # Filter errors only from any command
rtk log <file>          # Deduplicated logs with counts
rtk json <file>         # JSON structure without values
rtk deps                # Dependency overview
rtk env                 # Environment variables compact
rtk summary <cmd>       # Smart summary of command output
rtk diff                # Ultra-compact diffs
```

### Infrastructure (85% savings)
```bash
rtk docker ps           # Compact container list
rtk docker images       # Compact image list
rtk docker logs <c>     # Deduplicated logs
rtk kubectl get         # Compact resource list
rtk kubectl logs        # Deduplicated pod logs
```

### Network (65-70% savings)
```bash
rtk curl <url>          # Compact HTTP responses (70%)
rtk wget <url>          # Compact download output (65%)
```

### Meta Commands
```bash
rtk gain                # View token savings statistics
rtk gain --history      # View command history with savings
rtk discover            # Analyze Claude Code sessions for missed RTK usage
rtk proxy <cmd>         # Run command without filtering (for debugging)
rtk init                # Add RTK instructions to CLAUDE.md
rtk init --global       # Add RTK to ~/.claude/CLAUDE.md
```

## Token Savings Overview

| Category | Commands | Typical Savings |
|----------|----------|-----------------|
| Tests | vitest, playwright, cargo test | 90-99% |
| Build | next, tsc, lint, prettier | 70-87% |
| Git | status, log, diff, add, commit | 59-80% |
| GitHub | gh pr, gh run, gh issue | 26-87% |
| Package Managers | pnpm, npm, npx | 70-90% |
| Files | ls, read, grep, find | 60-75% |
| Infrastructure | docker, kubectl | 85% |
| Network | curl, wget | 65-70% |

Overall average: **60-90% token reduction** on common development operations.
<!-- /rtk-instructions -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivokun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ivokun)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
