## turbo-flow

> BRANCH=feat/search-feature

# CLAUDE.md — TurboFlow 4.0 / Ruflo v3.5

```
PROJECT_ID=rentamls
BRANCH=feat/search-feature
```

**Primary model: GLM-5.1** (via Coding Plan). Claude Opus on-demand for complex reasoning.
**Tech Stack:** Next.js 16.2.0, React 19, Prisma ORM, PostgreSQL (prod) / SQLite (dev), Railway

---

## BEHAVIORAL RULES

- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary — prefer editing existing files
- NEVER proactively create .md or README files unless explicitly requested
- NEVER save working files or tests to the root folder
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- ALWAYS use non-interactive shell flags — `cp -f`, `mv -f`, `rm -f`
- ALWAYS use `--json` flag with `bd` commands
- ALWAYS run tests before committing (if a test suite exists)
- After 3 failed approaches to the same problem — STOP and ask the human
- **NEVER merge to `main` without the Triple-Gate Merge Protocol**
- **NEVER force-push to `main` under any circumstances**
- Batch ALL related operations in a single message (todos, agent spawns, file ops, memory ops)

---

## WORK QUALITY

### Plan First
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Write detailed specs upfront to reduce ambiguity
- If something goes sideways, STOP and re-plan — don't keep pushing a broken approach
- Use plan mode for verification steps, not just building

### Autonomous Execution
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Go fix failing CI tests without being told how
- Use subagents liberally to keep main context window clean — offload research, exploration, and parallel analysis

### Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: implement the proper solution, not the quick patch
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### Self-Improvement Loop
- After ANY correction from the human, store the lesson: `bd remember "lesson/<topic>" "what went wrong and the rule to prevent it"`
- Write rules for yourself that prevent the same mistake
- Review stored lessons at session start: `npx ruflo@latest memory search -q "lesson" --limit 10`

---

## TRIPLE-GATE MERGE PROTOCOL

Any merge into `main` (`master`/`production`/`prod`/`release`) = 3 consecutive human confirmations. No agent may merge autonomously.

```
GATE 1 — "🔒 MERGE GATE 1/3: Merging [branch] → main. [changes, risk]. Confirm?"
GATE 2 — "🔒 MERGE GATE 2/3: Tests: [pass/fail]. Conflicts: [y/n]. Confirm?"
GATE 3 — "🔒 MERGE GATE 3/3: FINAL. Type 'yes' to execute."
```

Each gate = separate turn. Non-`yes` = abort. Does NOT apply to feature-to-feature merges.

**Destructive commands** (`git reset --hard`, `rm -rf`, `prisma migrate reset`, `DROP TABLE`): one confirmation. Format: `⚠️ DESTRUCTIVE: [command]. [consequence]. Confirm?`

**Rollback:** `git revert --no-commit HEAD` → test → commit → push (skips Triple-Gate) → tell human → `bd create` the bug → `bd remember "revert/[branch]" "cause"`

**Conflicts:** Never silently auto-resolve. Simple: resolve + show. Complex: show both sides, ask human.

---

## MODEL ROUTING

**GLM-5.1 (default):** 200K context, 131K max output. No tiered routing. Generate complete files in one pass for new files >100 lines. Give full task context and let it chain steps autonomously.

**Claude Opus (on-demand):** Check `[AGENT_BOOSTER_AVAILABLE]` / `[TASK_MODEL_RECOMMENDATION]` hooks. Tier 1: Agent Booster (WASM, $0) for simple transforms. Tier 2: Haiku. Tier 3: Sonnet/Opus. Hard cap: $15/hr.

**Token optimization:** Use `--json` on all `bd` commands. Prefer `bd ready --json` over `bd list`. Use `bd prime` sparingly. When context fills: `bd compact`. Offload verbose explanations to `bd comments` and `memory store` instead of chat.

**MCP tools:** CLI tools (bd, npx ruflo, npx gitnexus) work regardless of model. If MCP server calls fail with GLM-5.1, use the CLI equivalent. Run `npx ruflo@latest doctor --fix` to check MCP registration.

---

## BEADS (bd) — Project Truth

ALL issue tracking, decisions, blockers, and discovered work goes in Beads — never in markdown TODOs.

**CRITICAL:** Never directly read/write `.beads/issues.jsonl`. Command is `bd`, NOT `beads`. Run `bd sync flush` after batch ops. Every `bd create` MUST include `--description` — self-sufficient, as if the reader has never seen your conversation.

**Core commands:**

```bash
bd ready --json                          # What's unblocked? (session start)
bd update <id> --claim --json            # Claim before starting
bd comments add <id> "progress" --json   # Record findings mid-task
bd close <id> --reason "what+why" --json # Complete (prove it works first)
bd create "Title" --description="full context" -t bug|feature|task -p 0-4 --json
bd remember "key" "value"                # Persistent knowledge
bd compact                               # Summarize old beads (save context)
bd prime                                 # Full project state (use sparingly — heavy)
bd dolt push                             # Sync to remote (session end)
bd stale && bd orphans && bd lint        # Hygiene
```

**Creating issues — always include --description:**

```bash
# Bug (include: what, where, repro steps, expected vs actual)
bd create "Login redirect fails on expired JWT" \
  --description="Middleware returns 401 but client catches as generic error instead of routing to /login. File: src/middleware.ts:45. Repro: let token expire, click nav. Expected: /login redirect. Actual: blank 401." \
  -t bug -p 1 --json

# Feature (include: what, where to wire it, acceptance criteria)
bd create "Add bedroom/bathroom filter" \
  --description="Filter listings by min/max bedrooms/bathrooms. Wire to /api/listings query params. UI in SearchFilters component. Acceptance: AND logic with price filter, URL params persist on refresh." \
  -t feature -p 2 --json

# Sub-task: bd create "..." --parent bd-a3f8 -t task --json
# Discovered: bd create "..." --deps discovered-from:<id> --json
# Blocked: bd update <id> --status blocked --json + create blocker with --deps blocks:<id>
# Defer/Supersede/Escalate: bd defer|supersede|human <id> --json
```

**Types:** `bug` · `feature` · `task` · `epic` · `chore`  **Priorities:** `0` critical → `4` backlog

---

## RUFLO MEMORY & AGENTDB

### HNSW Pattern Store (Ruflo Memory)

```bash
npx ruflo@latest memory search -q "keywords" --limit 5                    # BEFORE solving
npx ruflo@latest memory store --namespace rentamls --key "area/fix" --value "what+why"  # AFTER solving
```

Aliases: `ruv-remember` · `ruv-recall` · `mem-search` · `mem-stats`

### AgentDB (MCP Tools)

**Search BEFORE writing fix code** — `agentdb_pattern-search` may already have a known solution.

Other tools: `agentdb_pattern-store` (store novel solutions) · `agentdb_context-synthesize` (combine context) · `agentdb_semantic-route` (pick approach) · `agentdb_hierarchical-store/recall` (knowledge trees)

---

## HOOKS & LEARNING

```bash
# Session start
npx ruflo@latest hooks session-start --session-id rentamls --start-daemon

# Before complex task
npx ruflo@latest hooks route "<task>" --include-explanation

# After task completion
npx ruflo@latest hooks post-task --task-id "<id>" --success true --store-results true

# After significant edits
npx ruflo@latest hooks post-edit --file "<file>" --train-patterns

# Session end
npx ruflo@latest hooks session-end --export-metrics true --persist-patterns true

# Diagnostics
npx ruflo@latest hooks intelligence stats     # What's been learned?
npx ruflo@latest hooks pretrain --depth deep  # Bootstrap (if 0 patterns)
npx ruflo@latest hooks worker dispatch --trigger audit  # Background audit
```

---

## SESSION WORKFLOW

**Start:**
1. `npx ruflo@latest hooks session-start --session-id rentamls --start-daemon`
2. `bd ready --json` + `bd list --type blocker --json`
3. `npx ruflo@latest memory search -q "lesson" --limit 10` — review past lessons
4. `npx ruflo@latest memory search -q "rentamls current state" --limit 5`

**Before non-trivial work:** `agentdb_pattern-search` → `mem-search` → `hooks route`

**During:** Claim beads → record progress in `bd comments` → `bd create` discovered work → `hooks post-edit` after significant changes

**After task:** Verify it works → `bd close` with proof → `agentdb_pattern-store` if novel → `memory store` → `hooks post-task` → `aqe-gate`

**End:**
1. `bd create` remaining work with full descriptions
2. `bd close` finished work
3. Quality gates: `npm test && npm run build`
4. `bd dolt push && git push` — **Work is NOT done until `git push` succeeds. YOU push.**
5. `npx ruflo@latest hooks session-end --export-metrics true --persist-patterns true`

---

## QUALITY & SECURITY

```bash
aqe-gate                                   # Full quality gate — required before merging
aqe-generate                               # Generate tests for new code
npx ruflo@latest security scan --depth full  # Security scan
npx ruflo@latest security cve --check        # CVE check
```

---

## PARALLEL EXECUTION

**Swarm:** Always hierarchical topology. Spawn ALL agents in ONE message. Use subagents to keep main context clean — one task per subagent for focused execution.

```bash
npx ruflo@latest swarm init --topology hierarchical --max-agents 8 --strategy specialized
Task("Architect", "Design...", "system-architect")
Task("Coder", "Implement...", "coder")
Task("Tester", "Write tests...", "tester")
```

**Worktrees:** One per agent. Each gets own `$DATABASE_SCHEMA`. Test before merge. `wt-clean` after.

```bash
git worktree add .worktrees/feat-a -b feat/feat-a
```

**Agent Teams:** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. Lead → up to 3 teammates (depth 2). Sub-agents cannot merge to main. **Requires API key sourced in `~/.bashrc`** — verify with `echo $ANTHROPIC_API_KEY`.

---

## GITNEXUS

Run blast radius before editing shared symbols: `gitnexus_impact({target: "name", direction: "upstream"})`. HIGH/CRITICAL = warn human. Pre-commit: `gitnexus_detect_changes({scope: "staged"})`. Stale index: `npx gitnexus analyze`. Never find-and-replace symbols — use `gitnexus_rename`.

---

## STATUS HUD

After action responses (not pure conversation):

```
───────────────────────────────────
📍 Branch: <branch> · <N> files changed
🧪 Tests: <status> · ⚡ Model: GLM-5.1 | Opus
───────────────────────────────────
```

---

## ENVIRONMENT

**Required:** DATABASE_URL, JWT_SECRET, OPENROUTER_API_KEY, NEXT_PUBLIC_APP_URL
**Billing:** CONEKTA_PUBLIC_KEY, CONEKTA_PRIVATE_KEY, CONEKTA_WEBHOOK_SECRET, BILLING_ENCRYPTION_KEY
**Optional:** RESEND_API_KEY

## RAILWAY

Builder: DOCKERFILE (not NIXPACKS). healthcheckTimeout ≥ 5000ms. Dockerfile: node:20-slim, CMD runs migrations via `scripts/docker-start.sh`. If healthcheck hangs: remove it. 502 = check migrations. Build fails = TypeScript errors.

## COMMON FIXES

Next.js 16 async params: `{ params }: { params: Promise<{ id: string }> }` then `const { id } = await params`

Prisma sync: `cp prisma/schema.prisma prisma/schema.prod.prisma` after every schema change.

## CURRENT WORK: SEARCH (feat/search-feature)

Phase 0 in progress: ✅ middleware fix, ✅ pg_trgm migration created, ⏳ run migration + verify.
Phases: 0→pg_trgm · 1→filters+UI · 2→tsvector · 3→cursor pagination · 4→faceting+autocomplete.
Docs: `docs/search/PR-REQUIREMENTS.md` · `docs/search/ADR-SEARCH-TECHNOLOGY.md` · `docs/search/IMPLEMENTATION-ROADMAP.md`

---
> Source: [marcuspat/turbo-flow](https://github.com/marcuspat/turbo-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
