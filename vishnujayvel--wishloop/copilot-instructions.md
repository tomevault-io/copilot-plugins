## wishloop

> |


# Wishloop v2

Thin session manager for Loki Mode. Classify work, spec it via OpenSpec CLI, configure and launch Loki, monitor the session, babysit the PR, loop.

## Prerequisites

- **openspec CLI** — `openspec` (spec generation)
- **loki CLI** — `loki start` (autonomous execution, requires `--dangerously-skip-permissions`)
- **gh CLI** — `gh issue create` / `gh issue list` (bug filing and feedback loop)
- **Worktrunk** *(recommended)* — `brew install worktrunk && wt config shell install` (worktree lifecycle, merge automation). Optional — the skill falls back to manual git worktrees when not installed.

## Design Principles

| Principle | Rule |
|-----------|------|
| **P1: Configure, Don't Reimplement** | If Loki has a capability, configure it via env vars, flags, or CLAUDE.md. Never rewrite it. |
| **P2: Monitor, Fix Post-Session** | Lightweight alive/dead/stalled check during session. Fix gaps after. |
| **P3: Match Session Type to Work** | `loki start --parallel` for features. `loki run #N --pr` for single bugs. `loki quick` for review fixes. |
| **P4: Strong Proposals, Thin Wrapper** | Quality of the Loki session = quality of the proposal. Invest in enrichment. |
| **P5: Upstream When Generic** | If a Wishloop feature is generic, contribute upstream. Plan B after 2 weeks. |
| **P6: External Interface Only** | Wishloop calls ONLY Loki's public CLI. NEVER internal run.sh functions. Zero `.loki/` file coupling. |

---

## Drain Mode

Autonomous loop that processes ALL open GitHub issues until the backlog is empty. Wraps v2 Steps 1-6 in a fetch-prioritize-launch-monitor-babysit cycle.

### Invocation

```bash
bash <skill-path>/scripts/drain.sh                          # drain all open issues
bash <skill-path>/scripts/drain.sh --label bug              # drain only bugs
bash <skill-path>/scripts/drain.sh --label refactor --max-iterations 3
```

Or via the skill trigger: `wishloop --drain` / `wishloop --drain --label bug`

### How It Works

```text
FETCH (gh issue list) → STOP? (zero = exit) → PRIORITIZE → RESUME? →
  PICK → CLASSIFY → LAUNCH (loki run #N --pr) →
  MONITOR (loki status --json, 5min polls) →
  POST (docs, learnings, archival) →
  BABYSIT (CodeRabbit → loki quick → merge) →
LOOP
```

### Key Design: GitHub Issues ARE the State

No local backlog file. GitHub Issues are the single source of truth:
- Open issues = work remaining
- Closed issues = work done  
- Issue labels = priority and filtering
- PR state = current work-in-progress

### Session Resumption

On re-invoke, drain mode checks:
1. `.wishloop/state.json` — what step were we in?
2. `gh pr list --state open` — if any, resume at BABYSIT
3. `loki status --json` — if running, resume at MONITOR
4. If neither, fetch open issues and start fresh

### Monitoring (via `loki status --json`)

The drain loop polls `loki status --json` every 5 minutes to detect:
- `completed` → proceed to post-session
- `stopped` → proceed to post-session
- `running` → continue polling (with stall detection)
- `unknown` → attempt `loki resume`, then crash recovery

### Emergency Stop (`loki stop`)

If the session is stalled (iteration unchanged for 15+ minutes), drain mode runs:
```bash
loki stop     # Kill stalled session
loki resume   # Restart from last checkpoint
```

### Priority Tiers

```text
critical > bug > auto-detected > refactor > enhancement > documentation
Within same tier: smaller effort first (body length heuristic)
```

### Cumulative Summary (on exit)

```text
=== Drain Complete ===
Iterations: 3 | Duration: 47 min | Commits: 12 | Files: 8
Issues at start: 5 | Resolved: 4 | Filed: 1 | Remaining: 2
Loki sessions: 3 | PRs merged: 4 | Quick fixes: 2
```

---

## Step 1: Intake

Classify the work request into a type and route accordingly.

| Signal | Work Type | Spec? | Loki Session Type |
|--------|-----------|-------|-------------------|
| "build X from scratch", greenfield | **Greenfield** | Full OpenSpec init | `loki start --parallel` |
| "add X", "integrate Y", new feature | **Feature** | `openspec new change` | `loki start --parallel` |
| "refactor", "migrate from X to Y" | **Refactor** | `openspec new change` (MODIFIED) | `loki start --parallel` |
| "fix #N", single issue | **Bug fix (single)** | None | `loki run #N --pr` |
| "fix these bugs", bug list | **Bug batch** | Batched changes | `loki start --parallel` per wave |
| "add tests", "E2E coverage" | **Testing** | Test-focused change | `loki start --parallel` |
| "write docs", "API docs" | **Documentation** | None | `loki docs generate` (no session) |
| "design X", "architecture for Y" | **Architecture** | Proposal + design only | None |
| "research", "spike", "evaluate" | **Research** | None | None |
| "audit X", "review Y for Z" | **Audit** | None | None |
| "rethink the X", "brainstorm" | **Product Thinking** | None | None |

**Non-code work types** (Architecture, Research, Audit, Documentation, Product Thinking): Load the matching template from `templates/` and skip to output. No Loki session needed.

| Work Type | Template |
|-----------|----------|
| Architecture (HLD) | `templates/hld.md` |
| Architecture (LLD) | `templates/lld.md` |
| Research | `templates/research.md` |
| Audit | `templates/audit.md` |
| Documentation | `templates/docs.md` |

---

## Step 2: Spec + Enrich

For code work types, produce the input that Loki needs.

### 2a. Generate spec

**Greenfield:** `openspec init --tools claude && openspec new change <name>`

**Feature/Refactor/Testing:** `openspec new change <name>` — write `proposal.md` with problem, goals, scope, non-goals, testing strategy, and exit criteria (defines the completion promise).

**Bug fix (single):** No spec needed. `loki run #N --pr` imports the GitHub issue directly.

**Bug fix in specced domain:** If the bug touches code governed by an existing OpenSpec domain spec, use `loki start --openspec` instead.

**Bug batch:** Run the intelligent batching algorithm (see `references/batching-algorithm.md`), then generate a minimal proposal per wave.

Loki can start from just a proposal. Only generate full OpenSpec artifacts (design, specs, tasks) for brownfield modifications needing structured delta context.

Validate: `openspec validate <name>` — all artifacts must pass before proceeding.

### 2b. Enrich proposal

```bash
bash <skill-path>/scripts/enrich-proposal.sh <project-dir> <proposal-path>
```

Injects: relevant file paths + line numbers, CLAUDE.md rules, test file patterns, build/run/test commands, recent git history, tech stack summary.

### 2c. Proposal quality gate

Before launching Loki, verify the proposal has:
- [ ] At least one file path reference
- [ ] At least one acceptance criterion
- [ ] A testing strategy
- [ ] The project's build command

Do not launch Loki with an incomplete brief — thin proposals cause wasted discovery time.

---

## Step 3: Configure

Before launching Loki, set the environment and clean state.

### 3a. Clean stale worktrees

```bash
git worktree prune 2>/dev/null
```

**Do NOT `rm -rf .loki/`.** Loki's initialization is additive — it preserves valid state. Destroying `.loki/` deletes checkpoints needed for `loki resume`.

### 3b. Pre-launch validation

```bash
loki doctor    # Verify prerequisites, skill symlinks, provider availability
```

If `loki doctor` fails: print diagnostic output. Do not proceed until doctor passes.

### 3c. Session configuration

Source the static config and compose the completion promise dynamically:

```bash
# Source static process config
set -a && source .wishloop/loki.env && set +a

# Extract task-specific exit criteria from the proposal
TASK_CRITERIA=""
if [ -f "$PROPOSAL_PATH" ]; then
  TASK_CRITERIA=$(sed -n '/## Exit Criteria/,/^## /p' "$PROPOSAL_PATH" | grep -E '^\s*[-*]' | sed 's/^[\s*-]*//' | tr '\n' '; ' | sed 's/; $//')
fi

# Compose full promise: task criteria + process bar
if [ -n "$TASK_CRITERIA" ]; then
  export LOKI_COMPLETION_PROMISE="${TASK_CRITERIA}. THEN: ${WISHLOOP_PROCESS_BAR}"
else
  export LOKI_COMPLETION_PROMISE="${WISHLOOP_PROCESS_BAR}"
fi
```

This ensures every session knows both WHAT to build (from the proposal) and HOW to deliver it (PR + review + respond).

### 3d. CLAUDE.md preparation

Ensure the project's CLAUDE.md contains:
- Build/test/lint commands
- Project conventions
- Any OpenSpec spec references
- Learnings format instruction and PR review handling:

```markdown
## Loki Session Rules

### Learnings
When you encounter errors, unexpected behavior, or learn something non-obvious,
ALWAYS update CONTINUITY.md's "## Mistakes & Learnings" section with bullet points:
- **What Failed:** [specific error]
- **Why It Failed:** [root cause]
- **How to Prevent:** [concrete action]

### PR Review Handling
After creating a PR, you MUST:
1. Wait for CodeRabbit to post its review (poll `gh pr checks` every 90 seconds)
2. Read all review comments: `gh pr view <N> --comments`
3. For each comment: fix the issue in code, then reply to the comment thread
   with what you changed: `gh pr review <N> --comment --body "Fixed: <description>"`
4. Push all fixes, then wait for CodeRabbit to re-review
5. Only declare completion when all review threads are resolved
```

### 3e. Worktrunk configuration (if available)

If `wt` is on PATH and no `.worktrunk.toml` exists, generate one with detected build system hooks.

### 3f. Checkpoint commit

```bash
git add -A && git commit -m "checkpoint: prep for loki session (<change-name>)"
```

Commit spec files, enriched proposal, and CLAUDE.md changes BEFORE creating worktrees.

### 3g. Launch

| Work Type | Launch Command |
|-----------|---------------|
| Feature/Greenfield/Refactor | `wt switch -c -x "loki start --parallel --openspec openspec/changes/<name>" <name>` |
| Bug fix (single) | `loki run #N --pr` |
| Bug batch wave | `wt switch -c -x "loki start --parallel --openspec openspec/changes/wave-N" wave-N` |

**Background fallback (no Worktrunk):**
```bash
nohup loki start --parallel --openspec openspec/changes/<name> > /tmp/loki-<name>.log 2>&1 &
LOKI_PID=$!
echo "Loki PID: $LOKI_PID"
```

If launch fails: check `/tmp/loki-<change>.log`. If Loki exits within 30 seconds, log the error.

**Initialize state tracking:**
```bash
echo '{"step":"session","change":"<name>","pid":PID,"startedAt":"ISO"}' > .wishloop/state.json
```

---

## Step 3.5: Monitor

While the Loki session runs, Wishloop monitors session health via Loki's public CLI.

**For Worktrunk sessions (`wt switch -c -x`):** Worktrunk blocks until the command completes. Check exit code: 0 = completed, non-zero = crashed.

**For background sessions:** Run the monitoring script every 5 minutes:

```bash
bash <skill-path>/scripts/gardening-check.sh <project-dir> <change-name>
# Or via /loop: /loop 5m bash <skill-path>/scripts/gardening-check.sh . my-change
```

The monitoring script uses `loki status --json` to check session health:

```bash
STATUS_JSON=$(loki status --json 2>/dev/null)
SESSION_STATUS=$(echo "$STATUS_JSON" | jq -r '.status')
ITERATION=$(echo "$STATUS_JSON" | jq -r '.iteration')
```

| Status | Meaning | Action |
|--------|---------|--------|
| `completed` | Session finished | Proceed to Step 4 |
| `stopped` | Stopped (by user or error) | Check logs, proceed to Step 4 |
| `running` | Still working | Continue polling |
| `unknown` | No active session | Check PID, may have crashed |

**Stall detection:** If `iteration` hasn't changed across 3 consecutive polls (15 minutes), emit a warning.

**Crash recovery:**
1. Log crash with error details
2. Clean orphaned worktrees: `git worktree prune`
3. Update `.wishloop/state.json` with `{"status": "crashed", "error": "..."}`
4. Suggest: `loki resume` (continue from last checkpoint)
5. Or: restart from Step 3 (clean launch)

**Emergency stop:** `loki stop` — kills a stalled session immediately.

### Wishloop Status Dashboard

When checking session status, display this combined view showing what Wishloop configured and how Loki is progressing:

```bash
# Gather data
STATUS_JSON=$(loki status --json 2>/dev/null)
STATE_JSON=$(cat .wishloop/state.json 2>/dev/null)
PROMISE=$(echo "$LOKI_COMPLETION_PROMISE" | fold -w 60)

# Display
echo "╔══════════════════════════════════════════════════════════╗"
echo "║  WISHLOOP STATUS                                        ║"
echo "╠══════════════════════════════════════════════════════════╣"
echo "║  Change:  $(echo "$STATE_JSON" | jq -r '.change // "unknown"')"
echo "║  Step:    $(echo "$STATE_JSON" | jq -r '.step // "unknown"')"
echo "║  Started: $(echo "$STATE_JSON" | jq -r '.startedAt // "unknown"')"
echo "╠══════════════════════════════════════════════════════════╣"
echo "║  LOKI SESSION                                           ║"
echo "║  Status:    $(echo "$STATUS_JSON" | jq -r '.status')"
echo "║  Phase:     $(echo "$STATUS_JSON" | jq -r '.phase')"
echo "║  Iteration: $(echo "$STATUS_JSON" | jq -r '.iteration')"
echo "║  Tasks:     $(echo "$STATUS_JSON" | jq -r '.task_counts.completed')/$(echo "$STATUS_JSON" | jq -r '.task_counts.total') completed"
echo "║  PID:       $(echo "$STATUS_JSON" | jq -r '.pid')"
echo "╠══════════════════════════════════════════════════════════╣"
echo "║  COMPLETION PROMISE                                     ║"
echo "║  $PROMISE"
echo "╠══════════════════════════════════════════════════════════╣"
echo "║  PROPOSAL                                               ║"
echo "║  $(head -1 "$PROPOSAL_PATH" 2>/dev/null || echo 'N/A')"
echo "║  Exit criteria: $(grep -c '^\s*[-*]' <(sed -n '/## Exit Criteria/,/^## /p' "$PROPOSAL_PATH" 2>/dev/null) 2>/dev/null || echo '0') items"
echo "╠══════════════════════════════════════════════════════════╣"
echo "║  PR STATUS                                              ║"
echo "║  $(gh pr list --json number,title,state --jq '.[0] | "#\(.number) \(.title) [\(.state)]"' 2>/dev/null || echo 'No open PR')"
echo "╚══════════════════════════════════════════════════════════╝"
```

This dashboard is displayed:
- At the start of each gardening check (Step 3.5)
- When the user asks for status
- In the cumulative summary at loop exit

**Action directives from monitoring:**

| Directive | Meaning | Action |
|-----------|---------|--------|
| `ACTION: SESSION_COMPLETE` | Loki completed successfully | Stop monitoring. Run Step 4 immediately. |
| `ACTION: SESSION_STALLED` | No progress for 15+ minutes | Warn user. Consider `loki stop` + `loki resume`. |
| `ACTION: SESSION_CRASHED` | Session died unexpectedly | Log error. Suggest `loki resume` or clean restart. |

---

## Step 4: Post-Session

After Loki's session ends (completion promise fulfilled or max iterations):

### 4a. Learnings verification

Loki's run.sh automatically calls its extraction pipeline at session end. Wishloop verifies it produced output:

```bash
MISTAKES=$(wc -l < ~/.loki/learnings/mistakes.jsonl 2>/dev/null || echo "0")
if [ "$MISTAKES" -le 1 ]; then
  # Loki's extraction found nothing — fall back to interactive extraction
  # Prompt user: What surprised you? What did Loki get wrong? New patterns?
  # Write to ~/.local/share/wishloop/learnings.json
fi
```

**Time-boxed exception:** This fallback exists because Loki's extraction pipeline currently produces empty output. Step 3d's CLAUDE.md instruction is designed to fix the root cause. Remove the fallback after 3 consecutive sessions produce JSONL with > 1 line.

### 4b. Documentation validation

```bash
loki docs check 2>/dev/null
```

If exit code is 1 (docs stale/missing) AND `--parallel` was used:
```bash
loki docs generate    # Fallback: generate full doc suite
```

### 4c. Run archival

```bash
bash <skill-path>/scripts/capture-run.sh <project> <change> <checkpoint-hash> <start-time> <pid>
```

Records: commit count, files changed, build/test results, duration, bugs found.

### 4d. Independent build/test verification

Run the project's own build and test commands:
```bash
npm run build && npm test    # or equivalent
```

If build or tests fail: file a GitHub issue with details. Proceed to Step 5 — the babysitter handles fixes.

---

## Step 5: PR Babysitter

After the Loki session creates a PR:

### 5a. Wait for CodeRabbit

Poll `gh pr checks <N>` every 90 seconds. Detect "Currently processing" in recent comments.

**Timeout:** If CodeRabbit hasn't responded within 10 minutes, proceed with `loki ci --pr` as the sole quality gate.

### 5b. Address review comments

When unresolved review threads are detected:

```bash
PR_BRANCH=$(gh pr view <N> --json headRefName --jq '.headRefName')
git checkout "$PR_BRANCH"

loki quick "Read and address all review comments on PR #<N>. \
  Run 'gh pr view <N> --comments' to see them. \
  Fix each issue, commit, and push."
```

If `loki quick` fails (3 iterations exhausted, unresolved threads remain):
1. Generate a focused proposal describing remaining comments
2. Launch `loki start` on the same PR branch with `LOKI_GITHUB_PR=false`
3. **Circuit breaker:** After 3 escalations for the same PR, pause and request user input.

**Multiple PRs (bug batch):**
```bash
bash <skill-path>/scripts/pr-babysitter.sh once
```

### 5c. Final quality gate

```bash
loki ci --pr --fail-on critical,high
```

Exit codes: 0 = pass, 1 = findings exceed threshold, 2 = error. On 1: file issue and continue. On 2: log warning, proceed.

### 5d. Merge

```bash
# With Worktrunk:
wt merge main

# Without:
gh pr merge <N> --squash --delete-branch
```

Auto-merge criteria: CI passes, no unresolved blocking comments, no merge conflicts. Only pause for user input when blocking comments require design decisions.

---

## Step 6: Loop

After PR is merged:

1. `openspec archive <change>` (if OpenSpec was used)
2. Clean up: `git worktree prune`
3. Fetch open issues: `gh issue list --state open --json number,title,body,labels`
4. If zero open issues: print "Pipeline clean" and exit
5. Run intelligent batching algorithm (see `references/batching-algorithm.md`)
6. Generate proposal for Wave 1
7. Return to Step 2

**Exit conditions:** Zero open issues, max iterations reached (default 5), or user types "stop."

**Cooldown:** 2 minutes between iterations. User can type "stop," "skip," or provide steering input.

### Cumulative summary (on exit):
```text
=== Wishloop Complete ===
Iterations: {N} | Duration: {M} min | Commits: {C} | Files: {F}
Issues at start: {N} | Filed: {N} | Resolved: {N} | Remaining: {N}
Learnings captured: {N}
```

---

## Loki Commands Used

| Command | When | Purpose |
|---------|------|---------|
| `loki doctor` | Pre-launch (Step 3) | Validate prerequisites and skill symlinks |
| `loki start --parallel` | Launch (Step 3g) | Feature/greenfield/refactor sessions |
| `loki start` (no parallel) | Escalation (Step 5b) | Fix complex review comments on existing PR branch |
| `loki run #N --pr` | Launch (Step 3g) | Single bug fix, issue-driven, lightweight |
| `loki quick "task"` | PR babysitter (Step 5b) | Fix review comments, 3 iterations max |
| `loki status --json` | Monitoring (Step 3.5) | Structured health check: status, iteration, task counts |
| `loki resume` | Crash recovery (Step 3.5) | Resume from last checkpoint |
| `loki ci --pr` | Pre-merge (Step 5c) | Final quality gate |
| `loki docs check` | Post-session (Step 4b) | Verify doc coverage |
| `loki docs generate` | Post-session fallback (Step 4b) | If docs stream didn't fire |
| `loki stop` | Emergency (Step 3.5) | Kill stalled session |

## State Tracking

`.wishloop/state.json` tracks pipeline position for resumption across sessions:

```bash
# At Step 3g (launch):
{"step": "session", "change": "<name>", "pid": PID, "startedAt": "ISO"}

# At Step 3.5 (session completes):
{"step": "post-session", "change": "<name>", "completedAt": "ISO"}

# At Step 5 (babysitter):
{"step": "babysitter", "pr": N, "change": "<name>"}

# At Step 6 (loop):
{"step": "loop", "iteration": N}
```

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/drain.sh [--label <X>] [--max-iterations <N>] [--cooldown <seconds>]` | Autonomous drain loop: fetch, prioritize, launch, monitor, babysit, merge, repeat |
| `scripts/gardening-check.sh <dir> <change>` | Lightweight session monitoring via `loki status --json` |
| `scripts/capture-run.sh <dir> <change> <hash> <time> <pid>` | Generate run instance JSON |
| `scripts/enrich-proposal.sh <dir> <proposal-path>` | Auto-enrich proposal with project context (Step 2b) |
| `scripts/pr-babysitter.sh [once\|interval] [max-cycles]` | Monitor all open PRs, triage reviews, merge in order (Step 5) |

## Reference Documents

| Reference | When to read |
|-----------|-------------|
| `references/batching-algorithm.md` | Bug batch classification or Step 6 wave planning |
| `references/run-instance-schema.md` | Step 4c run capture or querying historical runs |
| `references/learnings-schema.md` | Step 4a learnings verification |
| `references/worktree-isolation.md` | Step 3g when another Loki session is already running |

## Canonical data location

Run records and learnings live at `~/.local/share/wishloop/` (XDG-compliant, outside project repos).

---
> Source: [vishnujayvel/wishloop](https://github.com/vishnujayvel/wishloop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
