## solve

> |


# /solve — Strategic Problem Solver (agent-teams + gstack)

This skill **must** run as an agent-teams orchestrator. The lead session does intake +
synthesis + rendering. All framework execution and research is delegated to named teammates
so phases run in parallel, peers challenge each other, and Six Hats spawns 6 concurrent
role-plays. Wired to gstack for telemetry, learnings, repo-mode, question tuning, and
continuous checkpoint commits.

---

## Step 0 — Self-check + gstack preamble (run first, fail fast)

```bash
_TEL_START=$(date +%s)
_SESSION_ID="$$-$_TEL_START"
mkdir -p ~/.gstack/sessions ~/.gstack/analytics .context/solve
touch ~/.gstack/sessions/"$PPID"

# ---------- hard requirements ----------
_FAILURES=()

# Skill installed globally?
if [ ! -d "$HOME/.claude/skills/solve" ]; then
  _FAILURES+=("/solve not installed at ~/.claude/skills/solve. Install:")
  _FAILURES+=("    bash <(curl -fsSL https://raw.githubusercontent.com/bajpainaman/solve/main/install)")
fi

# Agent teams flag enabled?
_AT_FLAG="${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-}"
if [ -z "$_AT_FLAG" ] || [ "$_AT_FLAG" = "0" ]; then
  if [ -f "$HOME/.claude/settings.json" ] && \
     python3 -c "import json,sys;d=json.load(open('$HOME/.claude/settings.json'));sys.exit(0 if d.get('env',{}).get('CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS')=='1' else 1)" 2>/dev/null; then
    echo "AGENT_TEAMS: enabled in settings.json (will apply on next claude restart if not yet active)"
  else
    _FAILURES+=("agent teams not enabled. Run installer or add to ~/.claude/settings.json:")
    _FAILURES+=('    {"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}')
  fi
fi

# claude version >= 2.1.32
_CC_VER=$(claude --version 2>/dev/null | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+' | head -1)
if [ -n "$_CC_VER" ]; then
  awk -v v="$_CC_VER" 'BEGIN{split(v,a,".");exit !(a[1]>2 || (a[1]==2 && a[2]>1) || (a[1]==2 && a[2]==1 && a[3]>=32))}' \
    || _FAILURES+=("claude $_CC_VER too old, need >= 2.1.32 for agent teams")
fi

if [ ${#_FAILURES[@]} -gt 0 ]; then
  printf '\n/solve cannot start. Issues:\n'
  printf '  - %s\n' "${_FAILURES[@]}"
  exit 1
fi
echo "SELFCHECK: ok"

# ---------- gstack preamble ----------
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true

# Spawned-session detection (suppresses interactive prompts)
[ -n "$OPENCLAW_SESSION" ] && SPAWNED_SESSION=true || SPAWNED_SESSION=false
echo "SPAWNED_SESSION: $SPAWNED_SESSION"

# User preferences (gstack-config)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_EXPLAIN_LEVEL=$(~/.claude/skills/gstack/bin/gstack-config get explain_level 2>/dev/null || echo "default")
[ "$_EXPLAIN_LEVEL" != "default" ] && [ "$_EXPLAIN_LEVEL" != "terse" ] && _EXPLAIN_LEVEL="default"
_QUESTION_TUNING=$(~/.claude/skills/gstack/bin/gstack-config get question_tuning 2>/dev/null || echo "false")
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || echo "off")
_CROSS_PROJ=$(~/.claude/skills/gstack/bin/gstack-config get cross_project_learnings 2>/dev/null || echo "false")
_CHECKPOINT_MODE=$(~/.claude/skills/gstack/bin/gstack-config get checkpoint_mode 2>/dev/null || echo "explicit")
_CHECKPOINT_PUSH=$(~/.claude/skills/gstack/bin/gstack-config get checkpoint_push 2>/dev/null || echo "false")

echo "PROACTIVE: $_PROACTIVE"
echo "EXPLAIN_LEVEL: $_EXPLAIN_LEVEL"
echo "QUESTION_TUNING: $_QUESTION_TUNING"
echo "TELEMETRY: $_TEL"
echo "CROSS_PROJECT_LEARNINGS: $_CROSS_PROJ"
echo "CHECKPOINT_MODE: $_CHECKPOINT_MODE"

# Repo mode (solo / collaborative)
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"

# Branch + analytics ping
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"

if [ "$_TEL" != "off" ]; then
  echo '{"skill":"solve","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}' \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
```

If `_UPD` shows `UPGRADE_AVAILABLE <old> <new>`: tell the user "/solve has an update available: <old> -> <new>. Run `solve --update` to refresh." Continue the run; don't block.

If `_UPD` shows `JUST_UPGRADED <from> <to>`: print "Running /solve v{to} (just updated)."

If `SPAWNED_SESSION=true`, the skill is running inside another orchestrator (OpenClaw, etc.):
- Do NOT use AskUserQuestion. Auto-choose recommended options.
- Skip Lake Intro, telemetry consent, routing-rules prompt, and Context Recovery prompts.
- Focus on completing the task and reporting via prose output.

---

## Step 0.5 — Regime Classification (MANDATORY GATE)

**This step gates everything else.** /solve is heavyweight: 5 long-lived teammates + 6 just-in-time hat teammates + 25 frameworks + parallel research. That investment is correct for **strategic decisions** and badly miscalibrated for everything else. This step routes the question to the right shape.

### The classifier

AskUserQuestion (skip if `SPAWNED_SESSION=true`; auto-choose `decision` in that case):

> Before /solve runs the full workflow, what shape is this question?
>
> - **A) Strategic decision** — I'm picking between options, committing to a path, deciding whether to build/migrate/launch X. There's a commitment at the end.
> - **B) Knowledge / research** — I want to understand X, learn about Y, get a synthesis of a topic. There's no specific commit at the end.
> - **C) Bug / incident** — something is broken, I need root cause and a fix.
> - **D) Pre-product idea** — I have an idea that doesn't exist yet, want to think it through before building.

Persist the regime to `state.json::regime` via `bin/state-rw write`.

### Branch behavior (NON-NEGOTIABLE)

#### A) Strategic decision
- Continue to Step 1.
- **The lead's next tool call after Step 1's preamble MUST be `TeamCreate`.** No reading framework spec files first. No inline framework writing. The team is mandatory.
- Proceed through Steps 1-16 normally.

#### B) Knowledge / research
- Do NOT spawn the team. Do NOT call TeamCreate.
- Acknowledge the misfit: "This looks like a knowledge question, not a strategic decision. /solve's full 25-framework workflow is overkill here."
- Offer two paths:
  1. Answer the question directly using available research tools (WebSearch, parallel-research, browse). One-turn synthesis. No team. No frameworks.
  2. Convert to a decision: "If you'd like to turn this into a /solve decision (e.g. 'should we build X to address this?'), restate as a commitment question."
- If user picks (1): produce a focused 1-2 page synthesis. Cite sources. End.
- If user picks (2): re-run Step 0.5 with the new framing.
- Either way: do NOT continue to Steps 1-16. Skip telemetry's "completed" event; log as `outcome: redirected`.

#### C) Bug / incident
- Do NOT spawn the team.
- Invoke `/investigate` via the Skill tool with the problem statement as context.
- /investigate is purpose-built for root cause; /solve would re-derive the same answer with 10x the cost.
- Log: `outcome: handoff_investigate`. Exit.

#### D) Pre-product idea
- Do NOT spawn the team.
- Invoke `/office-hours` via the Skill tool with the problem statement as context.
- /office-hours is the YC-style brainstorming workflow; /solve's 5-question intake assumes a real problem with stakeholders, which a pre-product idea doesn't have yet.
- Log: `outcome: handoff_office_hours`. Exit.

### Why this gate exists

Without Step 0.5, the lead can interpret /solve's "must run as agent-teams" as soft guidance and decide that 25 frameworks is overkill for a knowledge question. The lead then runs inline (subagents instead of TeamCreate), produces a synthesis, and the user gets neither the full /solve workflow nor a clean handoff. This step removes the ambiguity: by the end of Step 0.5, the workflow is either (a) committed to a full team run, (b) producing a knowledge synthesis directly, or (c) handed off to a more-fit skill.

### Kickoff guard (only applies if regime = decision)

After Step 0.5 confirms `regime = decision`, the lead is bound by a **kickoff guard**:

- **Forbidden**: reading any `frameworks/<name>.md` file in the lead's own context
- **Forbidden**: writing any `$RUN_DIR/frameworks/<name>.md` file directly
- **Forbidden**: spawning subagents via the Agent tool for framework execution (subagents can't message peers; the team needs peer messaging)
- **Required**: the lead's next tool call after preamble (Step 1) and intake (Steps 4-5) is `TeamCreate`. Period.

If the lead finds itself reading a framework spec file or writing framework analysis directly, that's a kickoff-guard violation. Halt, restart the workflow at Step 6 (Create the Team), spawn the team, delegate to teammates.

### Confidence rubric impact

- Step 0.5 ran and `regime = decision` confirmed: +0 (baseline; expected behavior)
- Step 0.5 ran and `regime = knowledge|bug|idea` (handoff or redirect): N/A (rubric doesn't apply; /solve didn't run)
- Step 0.5 was skipped (SPAWNED_SESSION + auto-decision): +0 (baseline; trust orchestrator)

---

## Step 0.55 — Tier Pick (run only if regime = decision)

A full `Deep` run (all 25 frameworks) takes ~50-70 min. Most strategic decisions don't need that depth. Step 0.55 lets the user pick the right tier.

Skip this step if:
- `regime ≠ decision` (Step 0.5 routed to a lighter path)
- `SPAWNED_SESSION = true` (default to `standard`)
- `SOLVE_PRESEED_TIER` env var is set (CLI shim's `--tier` flag bypassed the question)

Otherwise, AskUserQuestion:

> How deep should this /solve run go?
>
> - **Quick** — 8 frameworks (~15 min, ~$1-2). The load-bearing only: First Principles, Issue Tree, Inversion, Iceberg, Cynefin, Decision Matrix, Six Hats, Minto.
> - **Standard** — 16 frameworks (~30 min, ~$3-5). Quick + Abstraction Ladder, Zwicky, Productive Thinking, Connection Circles, Concept Map, Eisenhower, Second-Order, Confidence-Speed-Quality. **Default for most decisions.**
> - **Deep** — all 25 frameworks (~50-70 min, ~$5-15). Full workflow with Ishikawa, Conflict Resolution, Balancing/Reinforcing Loops, Impact-Effort, Ladder of Inference, OODA, Hard Choice, SBI added. Use for one-way-door decisions with high stakes.

Persist to `state.json::tier`. Each phase's teammate filters its framework list against the tier:

```bash
TIER="${SOLVE_PRESEED_TIER:-${USER_PICK:-standard}}"
~/.claude/skills/solve/bin/state-rw write "$RUN_DIR" tier "\"$TIER\""
echo "TIER: $TIER ($(~/.claude/skills/solve/bin/tier "$TIER" --count) frameworks)"
```

### Tier-specific budget defaults

If user picks tier but doesn't pass explicit `--budget` and Step 0.6 budget intake is using defaults:

```bash
read DEFAULT_TIME DEFAULT_TOKENS < <(~/.claude/skills/solve/bin/tier defaults "$TIER")
# DEFAULT_TIME in seconds, DEFAULT_TOKENS in $
```

| Tier | Default time | Default tokens |
|---|---|---|
| Quick | 15 min | $2 |
| Standard | 30 min | $5 |
| Deep | 60 min | $15 |

### How teammates use this

Each teammate's brief (definer, systems-thinker, decider) reads `state.json::tier` and filters its framework list:

```bash
TIER=$(~/.claude/skills/solve/bin/state-rw read "$RUN_DIR" tier | tr -d '"')
ALLOWED_FRAMEWORKS=$(~/.claude/skills/solve/bin/tier "$TIER" --define)  # or --systems / --decide
```

Frameworks not in the tier are skipped silently. The framework's entry-predicate evaluation includes a tier check at the top:

```
if framework NOT IN tier(state.tier): skip with reason "not in $TIER tier"
```

This means a Quick-tier run never spawns the conditional frameworks (Hard Choice, SBI, Conflict Resolution Diagram), never runs Ishikawa, etc. Faster wallclock, lower cost, with the trade-off that depth is reduced.

### Confidence rubric impact

Quick / Standard runs have an inherently lower max rubric (fewer +2's available):
- Quick: max 6/10 (Cynefin = simple/complicated, Decision Matrix gap, Six Hats agreement — only 3 multipliers possible from completed frameworks)
- Standard: max 8/10 (adds Second-Order downside check)
- Deep: max 10/10 (full check)

The Minto pyramid notes the tier in the Confidence section: "rubric 8/10 (Standard tier; Deep tier could push higher with Hard Choice + Ladder of Inference + Reinforcing Loop checks)."

---

## Step 0.6 — Budget Intake (run only if regime = decision)

A /solve run can spawn 11 concurrent teammates and 30+ research queries. That can burn $5+ and 30+ minutes. Step 0.6 caps both axes upfront and halts the run if either is exceeded.

Skip this step if:
- `regime ≠ decision` (Step 0.5 routed to a lighter path)
- `SPAWNED_SESSION = true` (orchestrator decides budget; default unlimited)

Otherwise, AskUserQuestion in a single batch with two questions:

> **Question 1: Time budget for this run?**
> - 5 minutes (rapid; minimal research, smallest team)
> - 15 minutes (standard quick run)
> - 30 minutes (default for most decisions)
> - 1 hour (deep research; full 25 frameworks comfortably)
> - Unlimited (no time cap)

> **Question 2: Token budget for this run?**
> - $1 (lean; mostly cached evidence + minimal Parallel.ai queries)
> - $5 (standard; soft-warns at $4)
> - $20 (deep research; aggressive parallel queries OK)
> - Unlimited (no spend cap)

Persist both as `state.json::budget`:

```bash
TIME_SECONDS=$(case "$TIME_CHOICE" in
  "5 minutes") echo 300 ;;
  "15 minutes") echo 900 ;;
  "30 minutes") echo 1800 ;;
  "1 hour") echo 3600 ;;
  *) echo "null" ;;
esac)
TOKENS_USD=$(case "$TOKEN_CHOICE" in
  "\$1") echo 1 ;;
  "\$5") echo 5 ;;
  "\$20") echo 20 ;;
  *) echo "null" ;;
esac)
~/.claude/skills/solve/bin/state-rw write "$RUN_DIR" budget "{\"time_seconds\":$TIME_SECONDS,\"tokens_usd\":$TOKENS_USD}"
echo "BUDGET: time=${TIME_SECONDS}s, tokens=\$${TOKENS_USD}"
```

### Enforcement protocol

The lead checks budget between framework boundaries (after each completed framework file). One bash call:

```bash
export RUN_DIR
export SOLVE_COST_FILE
export TEL_START="$_TEL_START"
~/.claude/skills/solve/bin/budget-track --check both --run-dir "$RUN_DIR"
```

Exit codes:
- `0` = under cap, continue normally
- `1` = warned (≥ 80% of either cap). SendMessage to all teammates: "Budget at 80%. Wrap up current framework; do not start new ones."
- `2` = halted (cap exceeded). The lead must:
  1. SendMessage shutdown to every teammate
  2. Render a partial report to `.context/solve-<slug>.md` with whatever frameworks completed
  3. Append a `## Halted At Budget` section explaining `verdict: halted_time | halted_tokens`, what was complete, what's missing
  4. Set `state.json::partial = true` so next run can resume
  5. Surface to user: `solve halted: <time/tokens> exceeded. Run with bigger budget: solve --budget time=1h tokens=$20 "<problem>"`
  6. TeamDelete; log telemetry with `outcome: halted_on_cap`

The CLI shim's `--budget` flag pre-seeds Step 0.6 answers (skipping the AskUserQuestion in that case).

### Backward compat

If state.json::budget is missing or both fields are null, all checks default to unlimited. Existing runs (started before v0.5.0) continue working without budget gating.

---

## Step 0.7 — Calibration Check + Audit (run only if regime = decision)

Skip if `regime ≠ decision` or `SPAWNED_SESSION = true`.

### Pending followup (HIGH-bucket outcome verification)

The calibration ledger at `~/.gstack/calibration.jsonl` accumulates one row per completed /solve run with `outcome: TBD`. The most-recent HIGH-bucket entry that's 3-30 days old gets a follow-up question this run:

```bash
PENDING=$(~/.claude/skills/solve/bin/calibration pending-followup)
if [ "$PENDING" != "{}" ]; then
  PENDING_SLUG=$(echo "$PENDING" | python3 -c "import json,sys;print(json.load(sys.stdin).get('slug',''))")
  PENDING_REC=$(echo "$PENDING" | python3 -c "import json,sys;print(json.load(sys.stdin).get('recommendation_summary',''))")
  PENDING_TS=$(echo "$PENDING" | python3 -c "import json,sys;print(json.load(sys.stdin).get('ts_completed_iso',''))")
  echo "PENDING_FOLLOWUP: slug=$PENDING_SLUG since=$PENDING_TS"

  # Auto-detect git evidence (hint, not decision)
  EVIDENCE=$(~/.claude/skills/solve/bin/calibration detect-shipped "$PENDING_SLUG")
fi
```

If `PENDING_SLUG` is non-empty, AskUserQuestion (skip if `SPAWNED_SESSION`):

> Last /solve run was HIGH bucket on `<PENDING_SLUG>` (`<PENDING_TS>`).
> Recommendation: `<PENDING_REC>`.
> `<if EVIDENCE.detected: "I see git commits matching this since <date>. Confirm shipped?">`
> How did it go?

Options:
- A) Shipped, success
- B) Shipped, mixed (partial wins, partial losses)
- C) Shipped, failure
- D) Not yet shipped, still in progress
- E) Aborted, didn't ship

Map answer to ledger:
```bash
OUTCOME=$(case "$ANSWER" in
  A) echo "success" ;;
  B) echo "mixed" ;;
  C) echo "failure" ;;
  D) skip_log_outcome=1; echo "" ;;   # leave as TBD
  E) echo "aborted" ;;
esac)
[ -z "${skip_log_outcome:-}" ] && \
  ~/.claude/skills/solve/bin/calibration log-outcome "$PENDING_SLUG" "$OUTCOME"
```

### Drift audit

After updating the outcome (or if no pending followup but ledger has ≥ 10 settled HIGH entries):

```bash
AUDIT=$(~/.claude/skills/solve/bin/calibration audit)
echo "$AUDIT"
DRIFT_DETECTED=$(echo "$AUDIT" | python3 -c "import json,sys;d=json.load(sys.stdin);print('1' if d.get('drift_detected') else '0')")
```

If `DRIFT_DETECTED = 1`:
1. Surface warning to user via prose (not AskUserQuestion; this is informational):
   > "Calibration drift detected. HIGH-bucket success rate is `<X>%` over your last `<N>` settled runs. The recommendations the math says are HIGH are not shipping cleanly. I've raised the HIGH threshold from `rubric ≥ 8` to `rubric ≥ <new>`. Treat this run's recommendation with extra skepticism."
2. The threshold raise is automatic and persisted to `~/.gstack/calibration-config.json`. User can revert later with `solve --calibrate reset`.
3. Set `state.json::drift_warning` to the warning text.
4. Minto framework reads this in Step 12 and surfaces it in the Dissent section.
5. gstack-learnings-log: `{"type": "calibration_drift", "high_success_rate_pct": X, "threshold_raised_to": <new>}`.

### Backward compat

First-time runs (empty ledger) skip both followup and audit. Each completed run appends a row in Step 16.

---

## Step 1 — Preamble (gstack pattern + slug + parallel + browse)

```bash
echo "SESSION_ID: $_SESSION_ID"

# Parallel.ai key (env wins; falls back to ~/.gstack/parallel-api-key)
_PARALLEL_KEY="${PARALLEL_API_KEY:-}"
[ -z "$_PARALLEL_KEY" ] && [ -f ~/.gstack/parallel-api-key ] && \
  _PARALLEL_KEY=$(tr -d '[:space:]' < ~/.gstack/parallel-api-key 2>/dev/null)
if [ -n "$_PARALLEL_KEY" ]; then
  export PARALLEL_API_KEY="$_PARALLEL_KEY"
  echo "PARALLEL: ready"
else
  echo "PARALLEL: not configured (degrading to WebSearch + browse)"
fi

# gstack browse (research layer 3)
_BROWSE_BIN="$HOME/.claude/skills/gstack/browse/dist/browse"
[ -x "$_BROWSE_BIN" ] && echo "BROWSE: ready" || echo "BROWSE: missing"

# Slug
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || SLUG=unknown
echo "SLUG: $SLUG"

# Cost ledger
export SOLVE_COST_FILE=".context/solve/.cost-$_SESSION_ID"
echo "0" > "$SOLVE_COST_FILE"

# Timeline: started
~/.claude/skills/gstack/bin/gstack-timeline-log \
  '{"skill":"solve","event":"started","branch":"'"$_BRANCH"'","session":"'"$_SESSION_ID"'"}' \
  2>/dev/null &
```

### 1a. Prior learnings (compounding wisdom)

```bash
if [ "$_CROSS_PROJ" = "true" ]; then
  ~/.claude/skills/gstack/bin/gstack-learnings-search --limit 10 --cross-project 2>/dev/null || true
else
  ~/.claude/skills/gstack/bin/gstack-learnings-search --limit 10 2>/dev/null || true
fi
```

If learnings are surfaced, incorporate them into pre-flight intake reasoning. When a framework
output later matches a past learning, write in the report:

> **Prior learning applied:** `<key>` (confidence N/10, from `<date>`)

This makes the compounding visible. The user should see /solve getting smarter on their problems
over time.

If `_CROSS_PROJ` is unset/false and this is a first run, AskUserQuestion (skip if `SPAWNED_SESSION`):

> /solve can search learnings from your other projects on this machine to find patterns
> that might apply here. Stays local. Recommended for solo developers; skip if you work on
> multiple client codebases where cross-contamination would be a concern.

Options:
- A) Enable cross-project learnings (recommended)
- B) Keep learnings project-scoped only

If A: `~/.claude/skills/gstack/bin/gstack-config set cross_project_learnings true`
If B: `~/.claude/skills/gstack/bin/gstack-config set cross_project_learnings false`

### 1b. Context recovery (prior /solve runs for this slug)

```bash
_PROJ="${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}"
if [ -d "$_PROJ" ]; then
  echo "--- RECENT /solve ARTIFACTS ---"
  find "$_PROJ" -maxdepth 1 -type d -name 'solve-*' 2>/dev/null | sort | tail -5
  [ -f "$_PROJ/timeline.jsonl" ] && grep -F '"skill":"solve"' "$_PROJ/timeline.jsonl" 2>/dev/null | tail -5
  echo "--- END ARTIFACTS ---"
fi
# Local prior reports for this project
ls -t .context/solve-*.md 2>/dev/null | head -3
```

If a prior /solve report exists for a similar slug, AskUserQuestion (skip if `SPAWNED_SESSION`):

> Found a prior /solve run on `<slug>` from `<date>`. Build on that prior analysis or start fresh?

Options:
- A) Build on prior (read into context, treat as additional evidence)
- B) Start fresh (ignore prior; full re-deconstruction)

If A: read the prior report into context as evidence. Keep `intake.problem_refined` aligned
with the prior framing where it still holds.

### 1c. Routing rules check (CLAUDE.md)

```bash
_HAS_ROUTING="no"
if [ -f CLAUDE.md ] && grep -q "/solve" CLAUDE.md 2>/dev/null; then _HAS_ROUTING="yes"; fi
echo "HAS_ROUTING: $_HAS_ROUTING"
```

If `HAS_ROUTING=no` AND `_PROACTIVE=true` AND not `SPAWNED_SESSION`, surface this once
(don't auto-edit CLAUDE.md):

> Tip: this project's CLAUDE.md doesn't mention /solve. Adding a routing rule like
> "Strategic decisions / problem framing → invoke /solve" makes future Claude Code
> sessions invoke /solve automatically when relevant.

---

## Step 2 — Resolve Input

The user invokes one of:
- `/solve` (no args) → AskUserQuestion: open with "describe the problem in 1-3 sentences"
- `/solve "<one-liner>"` → use as-is
- `/solve <path>` → if `<path>` exists and is a file, read it as the problem brief

Then:

```bash
SOLVE_SLUG=$(echo "$PROBLEM_RAW" | head -c 60 | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]\+/-/g; s/^-\|-$//g')
RUN_DIR=".context/solve/$SOLVE_SLUG"
REPORT=".context/solve-$SOLVE_SLUG.md"
TEAM_NAME="solve-$SOLVE_SLUG"
mkdir -p "$RUN_DIR/evidence" "$RUN_DIR/frameworks" "$RUN_DIR/diagrams"
```

---

## Step 3 — Resume Detection

```bash
if [ -f "$RUN_DIR/state.json" ]; then echo "PARTIAL_RUN: yes"; else echo "PARTIAL_RUN: no"; fi
```

If `PARTIAL_RUN=yes`, AskUserQuestion (or auto-choose B if `SPAWNED_SESSION`):
- A) Resume from saved state
- B) Restart fresh (archives prior to `$RUN_DIR.bak.<ts>`)

`bin/state-rw init "$RUN_DIR" "$SOLVE_SLUG" "$PROBLEM_RAW"` initializes state on fresh runs.

---

## Step 4 — Pre-flight Intake (5 questions, question-tuning aware)

For each AskUserQuestion below, if `_QUESTION_TUNING=true`, run preference check first:

```bash
~/.claude/skills/gstack/bin/gstack-question-preference --check "solve-intake-<slot>" 2>/dev/null
# AUTO_DECIDE → use recommended option, say:
#   "Auto-decided <slot> → <option> (your preference). Change with /plan-tune."
# ASK_NORMALLY → ask via AskUserQuestion
```

If `SPAWNED_SESSION`, auto-choose recommended options without asking.

Otherwise AskUserQuestion in a single batch:

1. **Stakeholders** — single-decider / small-team / multi-stakeholder / org-wide
2. **Time pressure** — now / this-week / this-month / no-deadline
3. **Reversibility** — cheap / costly / one-way-door
4. **Decision-maker** — you-alone / you-with-input / someone-else / unclear
5. **Success criteria** — measurable-metric / qualitative-signal / cant-articulate / TBD

After each answer, log via `gstack-question-log`:

```bash
~/.claude/skills/gstack/bin/gstack-question-log '{
  "skill":"solve","question_id":"solve-intake-<slot>","question_summary":"<short>",
  "category":"clarification","door_type":"two-way","options_count":4,
  "user_choice":"<key>","recommended":"<key>","session_id":"'"$_SESSION_ID"'"
}' 2>/dev/null || true
```

Then free-form: "Refine the problem statement in your own words (this becomes canonical)."

Persist to `$RUN_DIR/intake.json` via `bin/state-rw write "$RUN_DIR" intake '<json>'`.

**Domain auto-detect** from problem text:
- contains "contract|API|deploy|audit|gas" → DOMAIN=eng
- contains "users|feature|ship|MVP|PMF|GTM" → DOMAIN=product
- contains "career|company|should I" → DOMAIN=strategy
- else DOMAIN=general

---

## Step 5 — Output Format Pick (question-tuning aware)

Same wrapping. AskUserQuestion: pyramid / build-up / tldr (or auto if SPAWNED). Persist to state.

---

## Step 6 — Create the Team (KICKOFF GATE)

**This is the kickoff gate.** If you (the lead) reached Step 6 with `regime = decision` from Step 0.5, your **next tool call MUST be TeamCreate**. No reading framework files. No writing inline framework analysis. No spawning subagents for framework work. TeamCreate first, everything else after.

```text
TeamCreate(team_name="$TEAM_NAME", description="Solve: $PROBLEM_REFINED")
```

Team file: `~/.claude/teams/$TEAM_NAME/config.json`. Shared task list: `~/.claude/tasks/$TEAM_NAME/`.

### Kickoff-guard violations (and how to recover)

If you find yourself doing any of these BEFORE calling TeamCreate, you've violated the kickoff guard:

1. Reading `~/.claude/skills/solve/frameworks/<name>.md` in your own context
2. Writing `$RUN_DIR/frameworks/<name>.md` directly with the Write tool
3. Spawning Agent subagents (without `team_name`) for framework work
4. Producing inline analysis like "let me apply Cynefin..." or "the iceberg layers are..."

**Recovery:** halt the violating action immediately. State: "I violated the kickoff guard; restarting at TeamCreate." Then call TeamCreate. Any work you did inline is discarded; teammates redo it. The reason: each teammate runs in its own context window, which is the architectural value of agent teams (peers can disagree, adversary can attack, hats can roleplay independently). Inline work loses this.

### Why TeamCreate first

The skill's value is **adversarial-by-construction reasoning**. Six Hats only works because the 6 hats spawn in parallel, isolated. The continuous adversary only works because it's a separate teammate watching the framework directory. Definer→researcher peer messaging only works because they share a task list. None of this can be simulated by inline analysis from one agent.

If `_CHECKPOINT_MODE=continuous`, the lead and each teammate make WIP commits at logical boundaries (see Continuous Checkpoint Mode section below).

---

## Step 7 — Spawn Long-Lived Teammates (5)

Use the Agent tool with `team_name="$TEAM_NAME"` and a `name` for each role.

| Name             | subagent_type     | Role |
|------------------|-------------------|------|
| `researcher`     | `general-purpose` | Layered web research; populates `$RUN_DIR/evidence/` |
| `definer`        | `general-purpose` | Phase 1: 8 Define frameworks |
| `systems-thinker`| `architect`       | Phase 2: 5 Systems frameworks |
| `decider`        | `general-purpose` | Phase 3: 12 Decide frameworks (excluding Six Hats fan-out) |
| `adversary`      | `critic`          | Continuous devil's-advocate; writes to `$RUN_DIR/adversary.jsonl` |

Each teammate receives a self-contained brief that includes:
- Skill root (`~/.claude/skills/solve/`) so they can read framework specs
- Run directory (`$RUN_DIR`)
- Problem statement and `intake.json`
- Their phase + framework list
- Communication protocol (Step 8)
- **EXPLAIN_LEVEL** value (terse → no jargon glosses; default → first-use glosses on jargon list)
- **CHECKPOINT_MODE** value (continuous → WIP commits between frameworks)
- **REPO_MODE** value (solo → flag system-wide concerns proactively; collaborative/unknown → flag via SendMessage to lead, don't fix)

### Researcher brief (template)
```
You are `researcher` in team $TEAM_NAME for /solve.
Skill root: ~/.claude/skills/solve
Run dir: $RUN_DIR
Problem: $PROBLEM_REFINED
Intake: $RUN_DIR/intake.json
EXPLAIN_LEVEL: $_EXPLAIN_LEVEL
CHECKPOINT_MODE: $_CHECKPOINT_MODE
REPO_MODE: $REPO_MODE

Your job: layered parallel web research. For each task you claim:
  1. Read the task description (it names the framework or sub-question).
  2. Compose 3-7 parallel research queries.
  3. Use $HOME/.claude/skills/solve/bin/parallel-research to fan out.
     If PARALLEL_API_KEY is set, it uses Parallel.ai. Otherwise it returns
     fallback objects telling YOU to run WebSearch.
  4. For top-N candidate sources, run gstack browse for full-page extraction.
  5. Write evidence to $RUN_DIR/evidence/<task-id>.md (markdown summary +
     bullets + source URLs).
  6. SendMessage to the framework owner when their evidence is ready.
  7. TaskUpdate completed.

Cost discipline: increment $SOLVE_COST_FILE for each external API call.
If running total exceeds $5, SendMessage to lead asking for green-light.
TOOL CALL CAP (NON-NEGOTIABLE):
You have a counter file at $RUN_DIR/teammate-counters/<your-name>.json.
Before EVERY tool call, increment the matching counter. If new value >= 6,
do NOT make the call. Instead, SendMessage to lead with:
  {"type": "cap_violation", "tool": "<Bash|Read|Write|Edit|Glob|Grep|WebSearch|WebFetch>",
   "count": 6, "framework": "<current framework name or 'none'>"}
Lead halts the entire run on cap_violation. This protects the user's budget.

```

### Definer brief
```
You are `definer` in team $TEAM_NAME.
EXPLAIN_LEVEL: $_EXPLAIN_LEVEL
CHECKPOINT_MODE: $_CHECKPOINT_MODE

Phase 1 (Define) frameworks in dependency order:
  Wave 1B (no deps): abstraction-ladder, first-principles, issue-tree,
                     ishikawa, inversion-define
  Wave 1C (deps on 1B): zwicky-box (request 1 sub-fan from researcher per axis),
                        conflict-resolution-diagram (only if intake.stakeholders != "single-decider"),
                        productive-thinking

For each framework:
  1. Read $HOME/.claude/skills/solve/frameworks/<name>.md.
  2. Apply the method. Use evidence from $RUN_DIR/evidence/.
  3. Write output to $RUN_DIR/frameworks/<name>.md following the framework's Output Schema.
  4. End with "What This Means For The Decision".
  5. TaskUpdate completed.

EUREKA LOGGING (CRITICAL):
  When `first-principles` strips assumptions and surfaces a contrarian truth that
  contradicts the conventional approach (e.g. "we don't actually need a bridge,
  we need a settlement layer"), log it:

    jq -n --arg ts "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
          --arg skill "solve" \
          --arg branch "$(git branch --show-current 2>/dev/null)" \
          --arg slug "$SOLVE_SLUG" \
          --arg insight "ONE_LINE_SUMMARY" \
        '{ts:$ts,skill:$skill,branch:$branch,slug:$slug,insight:$insight}' \
        >> ~/.gstack/analytics/eureka.jsonl 2>/dev/null || true

  Mark eureka in the framework output with: **EUREKA:** <insight>
  This compounds across all gstack skills.

If you need research evidence: SendMessage to `researcher` describing the sub-question.
If `adversary` flags weakness in your output (score >= 7): read adversary.jsonl,
revise the framework file, SendMessage to `adversary` for re-check.

If CHECKPOINT_MODE=continuous: between framework files, commit:
  git add $RUN_DIR/frameworks/<just-written>.md && git commit -m \\
    "WIP: solve phase 1 - <framework> complete

[gstack-context]
Decisions: <one-line>
Remaining: <what's left in phase 1>
Skill: /solve
[/gstack-context]"
TOOL CALL CAP (NON-NEGOTIABLE):
You have a counter file at $RUN_DIR/teammate-counters/<your-name>.json.
Before EVERY tool call, increment the matching counter. If new value >= 6,
do NOT make the call. Instead, SendMessage to lead with:
  {"type": "cap_violation", "tool": "<Bash|Read|Write|Edit|Glob|Grep|WebSearch|WebFetch>",
   "count": 6, "framework": "<current framework name or 'none'>"}
Lead halts the entire run on cap_violation. This protects the user's budget.

```

### Systems-thinker brief
```
You are `systems-thinker` in team $TEAM_NAME.
EXPLAIN_LEVEL: $_EXPLAIN_LEVEL
CHECKPOINT_MODE: $_CHECKPOINT_MODE

Phase 2 (Systems) frameworks, in order:
  iceberg, connection-circles, concept-map,
  balancing-loop (only if connection-circles found a balancing cycle),
  reinforcing-loop (only if connection-circles found a reinforcing cycle).

Inputs: all of $RUN_DIR/frameworks/* from Phase 1 and $RUN_DIR/evidence/*.
Output: write each to $RUN_DIR/frameworks/<name>.md with mermaid diagrams.

CRITICAL TRIGGERS (SendMessage to lead immediately if these fire):
  - iceberg's "structures" layer reveals a bug-shape (named system component
    failing reliably) → tell lead to invoke /investigate (auto-handoff if
    PROACTIVE=true; suggest only if PROACTIVE=false)
  - iceberg's "mental models" layer reveals pre-product / no-PMF uncertainty
    → tell lead to invoke /office-hours
  - structures layer reveals an explicit architecture decision → mark
    "PLAN_ENG_REVIEW_AT_END=true" in $RUN_DIR/state.json

REPO_MODE handling:
  - solo: flag any anomaly you notice across the codebase (we own everything)
  - collaborative/unknown: only flag what's directly relevant to the problem;
    SendMessage to lead with anything outside scope

Context Health: write a [PROGRESS] line to lead between frameworks:
  "[PROGRESS] iceberg done (3 cycles seeded). connection-circles next."
TOOL CALL CAP (NON-NEGOTIABLE):
You have a counter file at $RUN_DIR/teammate-counters/<your-name>.json.
Before EVERY tool call, increment the matching counter. If new value >= 6,
do NOT make the call. Instead, SendMessage to lead with:
  {"type": "cap_violation", "tool": "<Bash|Read|Write|Edit|Glob|Grep|WebSearch|WebFetch>",
   "count": 6, "framework": "<current framework name or 'none'>"}
Lead halts the entire run on cap_violation. This protects the user's budget.

```

### Decider brief
```
You are `decider` in team $TEAM_NAME.
EXPLAIN_LEVEL: $_EXPLAIN_LEVEL
CHECKPOINT_MODE: $_CHECKPOINT_MODE

Phase 3 (Decide) frameworks, in this order:
  cynefin (FIRST — its result routes the rest, see frameworks/cynefin.md),
  eisenhower, impact-effort, decision-matrix, second-order,
  ladder-of-inference, ooda-loop,
  hard-choice (only if intake.reversibility = one-way-door OR
               top-2 decision-matrix options within 5%),
  confidence-speed-quality, sbi (only if interpersonal signals).

Six Hats: when you reach that step, SendMessage to lead to spawn 6 hat-teammates.
They write their files. You synthesize after they idle.

Minto runs LAST and consumes everything. See frameworks/minto.md for output schema.

When all 12 frameworks complete, SendMessage to lead:
  "decider: phase 3 complete, ready for synthesis"

CONFUSION PROTOCOL:
  If Cynefin classifies as "disorder" (no domain has sum >= 8 OR top-two within
  1 point), STOP. Don't pick the closest domain. SendMessage to lead with:
    "decider: Cynefin = disorder. Halt phase 3. Need re-run of intake on <gap>."
  Lead decides whether to halt + re-intake or push through with explicit LOW
  confidence.

Context Health: [PROGRESS] line per framework completion.
TOOL CALL CAP (NON-NEGOTIABLE):
You have a counter file at $RUN_DIR/teammate-counters/<your-name>.json.
Before EVERY tool call, increment the matching counter. If new value >= 6,
do NOT make the call. Instead, SendMessage to lead with:
  {"type": "cap_violation", "tool": "<Bash|Read|Write|Edit|Glob|Grep|WebSearch|WebFetch>",
   "count": 6, "framework": "<current framework name or 'none'>"}
Lead halts the entire run on cap_violation. This protects the user's budget.

```

### Adversary brief
```
You are `adversary` in team $TEAM_NAME.
Long-lived watcher. Continuous devil's-advocate.

Watch $RUN_DIR/frameworks/. For every new file:
  1. Read it.
  2. List the strongest 3 ways its conclusion could be wrong.
  3. Identify hidden assumptions that, if false, invalidate it.
  4. Score adversarial confidence 0-10 (10 = "I can break this easily").
  5. Append one JSON line per framework to $RUN_DIR/adversary.jsonl:
       {"framework":"<name>","attacks":[...],"assumptions":[...],"score":N,"ts":"..."}
  6. If score >= 7: SendMessage to the framework owner asking for revision.

Voice enforcement: if a framework output uses any of these AI words, flag it:
  delve, crucial, robust, comprehensive, nuanced, multifaceted, furthermore,
  moreover, additionally, pivotal, landscape, tapestry, underscore, foster,
  showcase, intricate, vibrant, fundamental, significant.
SendMessage to owner: "voice violation: <word> at <line>. Rewrite without it."

QUESTION AUTO-CONVERSION (NEW IN v0.5.0):
While reading framework outputs, scan for prose questions that should have
gone through AskUserQuestion. A line is a candidate question if:
  - It ends with '?'
  - It is in active voice (not buried inside a quote or markdown table)
  - It does NOT contain "this means" / "obviously" / "of course" (rhetorical)
  - It does NOT have <!-- skip-question-scan --> on the line
For each candidate, write a pending-questions file:
  $RUN_DIR/pending-questions/<framework>-<seq>.json
Schema: {"from": "<framework owner teammate>", "framework": "<name>",
         "question": "<exact text>", "context": "<surrounding 2 lines>"}
The lead polls $RUN_DIR/pending-questions/ between framework boundaries,
calls AskUserQuestion for each, relays the answer back via SendMessage,
then deletes the pending file.

TOOL CALL CAP (READ-EXEMPT):
You're exempt from the Read cap (you read every framework file). Counters
for Bash/Write/Edit/Glob/Grep/WebSearch/WebFetch still apply at 6.
SendMessage cap_violation to lead if you hit 6 of any non-Read tool.

Continue until lead sends `{"type":"shutdown_request"}`.
```

---

## Step 8 — Communication Protocol

- **Lead → teammate**: TaskUpdate with `owner: <name>`. Then SendMessage with briefing details.
- **Teammate → lead**: idle notifications automatic. SendMessage for blockers, trigger fires, cost-cap requests, CONFUSION protocol fires, `[PROGRESS]` updates.
- **Peer → peer**: SendMessage by name (`definer` → `researcher` for a sub-question).
- **Adversary → owner**: SendMessage when adversarial score >= 7 OR voice violation detected.

All teammates **must** mark tasks `completed` via TaskUpdate when done.

### Lead pending-questions polling (NEW IN v0.5.0)

Between framework boundaries, lead checks `$RUN_DIR/pending-questions/`:

```bash
if [ -d "$RUN_DIR/pending-questions" ]; then
  for q_file in "$RUN_DIR/pending-questions"/*.json; do
    [ -f "$q_file" ] || continue
    Q_FROM=$(python3 -c "import json;print(json.load(open('$q_file'))['from'])")
    Q_TEXT=$(python3 -c "import json;print(json.load(open('$q_file'))['question'])")
    Q_FRAMEWORK=$(python3 -c "import json;print(json.load(open('$q_file'))['framework'])")
    # Lead invokes AskUserQuestion with Q_TEXT, gets response
    # SendMessage to Q_FROM with the answer
    # rm "$q_file"
  done
fi
```

The adversary teammate writes these files when scanning framework outputs (see Adversary brief in Step 7).

### Lead budget polling (NEW IN v0.5.0)

Between framework boundaries, lead also runs:

```bash
~/.claude/skills/solve/bin/budget-track --check both --run-dir "$RUN_DIR"
```

Exit 1 (warned at 80%): SendMessage to all teammates to wrap up. Exit 2 (halted): trigger halt protocol from Step 0.6.

### Lead progress display (NEW IN v0.5.1)

Between framework boundaries, lead prints the current progress snapshot:

```bash
~/.claude/skills/solve/bin/progress --run-dir "$RUN_DIR"
```

This produces a 3-phase Unicode-bar block showing per-phase completion, adversary max score, budget burn, and the latest framework written. The user sees this in the conversation as a tool-call result, giving them live visibility into where the run is.

For terse prompts (`EXPLAIN_LEVEL=terse`), use `--plain` instead, which prints `[N/25] current-framework` on a single line.

Display rules:
- Print at the START of each phase (before any framework in that phase runs)
- Print after each framework completes
- Print at HALT (budget overrun or cap violation) for context
- Print FINAL snapshot before report rendering (Step 12)

---

## Step 8.5 — Tool-Call Cap Enforcement (NON-NEGOTIABLE GUARDRAIL)

Each teammate is capped at **5 calls of any single tool**. Hit 6 of any one tool = halt the entire /solve run.

### Why per-tool, not cumulative

A definer running 8 frameworks legitimately needs 8 Read calls (one per framework spec). A definer making 8 Bash calls is stuck in a loop. Per-tool caps catch the loop without penalizing legitimate work.

### Counter mechanics

Each teammate maintains a counter file at `$RUN_DIR/teammate-counters/<name>.json`:

```json
{"bash": 0, "read": 0, "write": 0, "edit": 0, "glob": 0, "grep": 0, "websearch": 0, "webfetch": 0}
```

**Before every tool call**, the teammate:
1. Reads its counter file
2. Increments the counter for the tool it's about to use
3. If the new value is ≥ 6: do NOT make the call. Instead, SendMessage to lead:
   ```
   {"type": "cap_violation", "tool": "Bash", "count": 6, "framework": "<current>"}
   ```
4. If under cap: write incremented counter back, make the call

### Lead's response to cap_violation

Receiving a cap_violation message triggers the halt protocol:
1. SendMessage shutdown to all teammates immediately
2. Render partial report at `.context/solve-<slug>.md` with whatever frameworks completed
3. Append a `## Halted On Cap Violation` section explaining which teammate hit which cap
4. Set `state.json::partial = true` and `state.json::halt_reason = "cap_violation_<tool>"`
5. Surface to user:
   > "/solve halted: teammate `<name>` hit 6 calls on `<tool>` while working on `<framework>`. This is the cap to prevent runaway loops. To continue, restart with: `solve --budget time=1h tokens=$20 --skip-frameworks <completed-list> '<problem>'`"
6. TeamDelete; log telemetry: `outcome: halted_cap_violation`

### Adversary exemption

The adversary teammate is exempt from the **Read cap only** (it watches every framework file as it's written, which legitimately requires 25+ reads). All other tool caps apply. The adversary's counter file omits the `read` field entry for tracking.

### Hat teammate caps (Phase 3)

Each hat teammate (white/red/black/yellow/green/blue) is short-lived (writes one file, idles). They get fresh counters per spawn. Hat caps still apply but rarely matter in practice.

### Verification at run start

Lead initializes counter files at Step 7 (after spawning teammates):

```bash
mkdir -p "$RUN_DIR/teammate-counters"
for name in researcher definer systems-thinker decider adversary; do
  echo '{"bash":0,"read":0,"write":0,"edit":0,"glob":0,"grep":0,"websearch":0,"webfetch":0}' \
    > "$RUN_DIR/teammate-counters/$name.json"
done
# Adversary is read-exempt; mark with sentinel
echo '{"bash":0,"read":-1,"write":0,"edit":0,"glob":0,"grep":0,"websearch":0,"webfetch":0}' \
  > "$RUN_DIR/teammate-counters/adversary.json"
```

(Read = -1 means "not tracked"; teammates check `if count != -1 && count >= 6`.)

---

## Step 9 — Populate the Shared Task List

```text
Phase 1 tasks (assign owner=definer):
  T-define-abstraction-ladder, T-define-first-principles, T-define-issue-tree,
  T-define-ishikawa, T-define-inversion, T-define-zwicky-box,
  T-define-conflict-res, T-define-productive-thinking

Phase 1 research tasks (owner=researcher):
  T-research-define-prior-art, T-research-define-failure-modes
  (extra T-research-zwicky-axis-N spawned as definer requests)

Phase 2 tasks (owner=systems-thinker, blockedBy phase-1 group):
  T-systems-iceberg, T-systems-connection-circles, T-systems-concept-map
  (T-systems-balancing-loop, T-systems-reinforcing-loop are conditional;
  systems-thinker creates them mid-run if cycles surface)

Phase 2 research tasks (owner=researcher):
  T-research-systems-analogous, T-research-systems-failure-modes-prior

Phase 3 tasks (owner=decider, blockedBy phase-2 group):
  T-decide-cynefin (must complete first within phase 3),
  T-decide-eisenhower, T-decide-impact-effort, T-decide-decision-matrix,
  T-decide-second-order, T-decide-ladder-of-inference, T-decide-ooda,
  T-decide-hard-choice, T-decide-confidence-speed-quality,
  T-decide-sbi, T-decide-minto

Phase 3 research tasks (owner=researcher):
  T-research-decide-base-rates, T-research-decide-prior-art

Continuous task (no owner; adversary self-claims):
  T-adversary-monitor (lead marks complete on shutdown)
```

---

## Step 10 — Six Hats Spawn (Phase 3 mid-flight)

When `decider` reaches Six Hats it SendMessages the lead. Lead spawns 6 teammates **in parallel**:

| Name              | subagent_type   | Hat & focus |
|-------------------|-----------------|-------------|
| `hat-white`       | general-purpose | Facts only, verifiable evidence |
| `hat-red`         | general-purpose | Gut, intuition, emotion |
| `hat-black`       | critic          | Risks, failure modes, devil's advocate |
| `hat-yellow`      | general-purpose | Justified optimism, upside |
| `hat-green`       | general-purpose | Lateral, alternative recommendations |
| `hat-blue`        | architect       | Process meta, what's missing |

Each gets the same brief plus its hat role from `frameworks/six-hats.md`. Each writes
`$RUN_DIR/frameworks/six-hats-<color>.md` and idles.

Once all 6 idle, the lead synthesizes (or asks `decider` to). SendMessage shutdown to each
hat teammate individually.

---

## Step 11 — Confidence Synthesis (lead, no team needed)

Compute three views from framework files. See [Confidence Math](README.md#confidence-math) for the rubric.

1. **Rubric (0-10)**, additive
2. **Bayesian posterior**, P(decision correct | evidence)
3. **Bucket**, HIGH / MEDIUM / LOW

Persist via `bin/state-rw write "$RUN_DIR" confidence '<json>'`.

---

## Step 12 — Render Report

```bash
~/.claude/skills/solve/bin/render-report \
  --slug "$SOLVE_SLUG" \
  --format "$FORMAT" \
  --run-dir "$RUN_DIR" \
  --output "$REPORT"
open "$REPORT" 2>/dev/null || true
```

---

## Step 13 — Cross-Skill Handoffs

If triggers fired during the run, dispatch via Skill tool. Behavior depends on `_PROACTIVE`:

| Trigger | If PROACTIVE=true | If PROACTIVE=false |
|---|---|---|
| `iceberg.structures = bug-shape` | Auto-invoke `/investigate` mid-run | Suggest only at end |
| `iceberg.mental_models = pre-product` | Auto-invoke `/office-hours` | Suggest only |
| `state.PLAN_ENG_REVIEW_AT_END = true` | Suggest `/plan-eng-review` at end | Suggest only |

Log every handoff to `$RUN_DIR/handoffs.jsonl`. After a handoff completes, write a review entry:

```bash
~/.claude/skills/gstack/bin/gstack-review-log '{
  "skill":"solve","trigger":"<which trigger>","handoff_to":"<skill>",
  "outcome":"<short>","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","slug":"'"$SOLVE_SLUG"'"
}' 2>/dev/null || true
```

---

## Step 14 — Shutdown Team

```text
SendMessage(to="researcher", message={"type":"shutdown_request"})
SendMessage(to="definer", ...)
SendMessage(to="systems-thinker", ...)
SendMessage(to="decider", ...)
SendMessage(to="adversary", ...)
# any leftover hat teammates
```

After all idle, **TeamDelete**. Lead does this; never a teammate.

---

## Step 15 — Learnings + Eureka Extraction

After the team shuts down, lead scans framework files for non-obvious insights worth remembering.

### Learnings (durable patterns)

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{
  "skill":"solve","type":"insight","key":"<short>",
  "insight":"<observed pattern>","confidence":N,"source":"observed",
  "slug":"'"$SOLVE_SLUG"'"
}' 2>/dev/null || true
```

0-5 learnings per run. Patterns to look for:
- "User consistently under-weights second-order effects"
- "Domain X consistently surfaces Cynefin = complex"
- "When stakeholders > N, conflict-resolution-diagram is decisive"

### Eureka (contrarian first-principles findings)

The definer logs eureka entries when first-principles atoms contradict conventional approaches. The lead surfaces a final eureka summary in the report:

```bash
# Read eureka entries written this run
grep -F "$_SESSION_ID" ~/.gstack/analytics/eureka.jsonl 2>/dev/null
```

If eureka entries exist, include them in the report's "Insights" appendix.

### Question Tuning offer

If `_QUESTION_TUNING=true` and any AskUserQuestion fired this run, offer:
> Tune any of these questions? Reply `tune: never-ask <id>`, `tune: always-ask <id>`,
> or `tune: ask-only-for-one-way <id>`.

User-origin gate: only act on `tune:` if it appears in the user's own current message.

---

## Step 16 — Cost + Telemetry (run last)

```bash
_COST=$(cat "$SOLVE_COST_FILE" 2>/dev/null || echo "0")
echo "Total API spend this run: \$$_COST"
if (( $(echo "$_COST > 5" | bc -l 2>/dev/null) )); then
  echo "WARN: exceeded \$5 soft cap"
fi

_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))

# Timeline: completed
~/.claude/skills/gstack/bin/gstack-timeline-log '{
  "skill":"solve","event":"completed",
  "branch":"'$(git branch --show-current 2>/dev/null || echo unknown)'",
  "outcome":"success","duration_s":"'"$_TEL_DUR"'",
  "session":"'"$_SESSION_ID"'","slug":"'"$SOLVE_SLUG"'","cost_usd":"'"$_COST"'"
}' 2>/dev/null || true

# Local analytics (gated by telemetry config)
if [ "$_TEL" != "off" ]; then
  echo '{"skill":"solve","duration_s":"'"$_TEL_DUR"'","outcome":"success","session":"'"$_SESSION_ID"'","cost_usd":"'"$_COST"'","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}' \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi

# Calibration ledger (NEW IN v0.5.0): append a TBD-outcome row for this run.
# Future runs ask the user about the outcome; the ledger drives drift detection.
if [ "$_REGIME" = "decision" ] && [ -n "${_BUCKET:-}" ]; then
  ~/.claude/skills/solve/bin/calibration log \
    "$SOLVE_SLUG" "$_SESSION_ID" "$_BUCKET" "${_RUBRIC:-null}" "${_POSTERIOR:-null}" \
    "${_RECOMMENDATION_SUMMARY:-(rendered)}" 2>/dev/null || true
fi

# Remote telemetry (opt-in)
if [ "$_TEL" != "off" ] && [ -x ~/.claude/skills/gstack/bin/gstack-telemetry-log ]; then
  ~/.claude/skills/gstack/bin/gstack-telemetry-log \
    --skill "solve" --duration "$_TEL_DUR" --outcome "success" \
    --used-browse "yes" --session-id "$_SESSION_ID" 2>/dev/null &
fi
```

---

## Plan Mode Safe Operations

If `/solve` is invoked in plan mode, treat the skill file as executable instructions, not
reference. The first AskUserQuestion is the workflow entering plan mode, not a violation.
Allowed because they inform the plan: gstack bin tools, parallel-research, gstack browse,
writes to `~/.gstack/`, writes to `$RUN_DIR/`. Call ExitPlanMode only after the workflow
completes (Step 16).

If a STOP point fires (e.g. CONFUSION protocol), stop. Do not continue or call
ExitPlanMode.

---

## Confusion Protocol

When evidence sorts poorly (Cynefin = disorder, stakeholder framing contradicts itself,
or the problem statement keeps mutating across phases), STOP. Name the ambiguity in one
sentence. Present 2-3 options with tradeoffs. Ask via AskUserQuestion (or SendMessage to
lead from a teammate). Do not silently auto-decide.

---

## Continuous Checkpoint Mode

If `_CHECKPOINT_MODE=continuous`, lead and teammates make WIP commits at logical boundaries:
- Lead: after Step 5 (intake + format locked), after Step 11 (confidence computed), after Step 12 (report rendered)
- Definer: between framework files, and after Wave 1B, Wave 1C
- Systems-thinker: between framework files
- Decider: between framework files

Commit format:
```
WIP: solve <phase> - <step or framework>

[gstack-context]
Decisions: <key choices made this step>
Remaining: <what's left in this phase>
Tried: <failed approaches worth recording> (omit if none)
Skill: /solve
[/gstack-context]
```

Rules: stage only intentional files (`$RUN_DIR/*`, never `git add -A`), do not commit
broken state, push only if `_CHECKPOINT_PUSH=true`.

If `_CHECKPOINT_MODE=explicit`, ignore this section.

---

## Context Health

During Phase 1 and Phase 2 (long phases with many frameworks), each teammate writes
`[PROGRESS] <line>` to lead between framework completions. Format:

```
[PROGRESS] iceberg done. 2 cycles surfaced. concept-map next.
```

If a teammate is looping on the same framework or same evidence, STOP and reassess.
Consider escalation to lead or to /context-save. Progress lines never mutate state.

---

## Repo Ownership

`REPO_MODE` controls how teammates handle issues outside the immediate problem:
- **solo**: teammates flag anything anomalous they notice (we own everything)
- **collaborative / unknown**: teammates only flag what's relevant; SendMessage lead with anything outside scope

Always flag anything that looks wrong with one sentence: what was noticed and its impact.

---

## Search Before Building (eureka pattern)

Before any framework commits to a recommendation, the agent applies the layered-research
ethos:
- **Layer 1** (tried-and-true): does an established workflow solve this?
- **Layer 2** (new-and-popular): is there a recent technique gaining traction?
- **Layer 3** (first-principles): if you stripped all conventions, what would the atoms imply?

When Layer 3 contradicts Layers 1+2, that's a eureka moment. Log it (see Step 15). Don't
suppress it; surface it in the framework output and let Decision Matrix weight it.

---

## Boil the Lake (completeness principle)

AI makes completeness cheap. Recommend complete lakes (every dimension, every framework,
every adversary attack examined). Flag oceans (rewriting the whole skill, multi-quarter
roadmaps). When framework outputs differ in coverage, include a `Completeness: N/10`
score in the framework output itself (see frameworks/*.md for per-framework rubrics).

A run with average framework completeness < 6 caps overall confidence at MEDIUM regardless
of rubric score.

---

## Writing Style

If `_EXPLAIN_LEVEL=default`, gloss curated jargon on first use per skill invocation, even
if the user pasted the term. Frame outputs in decision-impact terms (what changes for
the team, what they can act on, what risk evaporates). Short sentences, concrete nouns,
active voice.

If `_EXPLAIN_LEVEL=terse`, no glosses, no impact-framing layer, shorter responses.

User-turn override wins: if the user's current message asks for terse / no explanations,
skip glosses regardless of config.

### Jargon list (gloss on first use if EXPLAIN_LEVEL=default)

Cross-cutting:
- emergence, retrospective coherence, dominance, equifinality
- MECE (mutually exclusive, collectively exhaustive)
- Bayesian posterior, prior, likelihood
- second-order effect, third-order effect
- reinforcing loop, balancing loop, leverage point

Domain-specific:
- (eng) idempotent, race condition, deadlock, N+1, eventual consistency, CAP theorem
- (web3) bridge, OFT, executor, mempool, gas spike
- (product) PMF, GTM, wedge, demand reality, narrowest wedge
- (ops) one-way-door, two-way-door, runway

---

## Voice

Builder-to-builder. Direct, concrete. Name files, frameworks, commands explicitly. Lead
with the point.

No em dashes. No AI vocabulary: delve, crucial, robust, comprehensive, nuanced,
multifaceted, furthermore, moreover, additionally, pivotal, landscape, tapestry,
underscore, foster, showcase, intricate, vibrant, fundamental, significant.

The adversary teammate enforces this on every framework output. Voice violations get
flagged with line numbers.

The user has context the framework does not. Cross-framework agreement is a
recommendation, not a decision. The user decides.

---

## Completion Status Protocol

- **DONE**: all 25 frameworks ran, report rendered, team cleaned up.
- **DONE_WITH_CONCERNS**: completed but adversary max score >= 7, or completeness avg < 6.
- **BLOCKED**: research stack failed AND no fallback worked.
- **NEEDS_CONTEXT**: pre-flight intake revealed problem isn't well-defined OR Cynefin = disorder.

Format: `STATUS`, `REASON`, `ATTEMPTED`, `RECOMMENDATION`. Escalate after 3 failed attempts.

---

## Operational Self-Improvement

Before completing, if you discovered a durable project quirk or workflow improvement worth
5+ minutes next time, log it:

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{
  "skill":"solve","type":"operational","key":"SHORT_KEY",
  "insight":"DESCRIPTION","confidence":N,"source":"observed"
}' 2>/dev/null || true
```

Skip obvious facts and one-time transient errors.

---

## Limitations of agent-teams mode

- Experimental. Session resumption (`/resume`, `/rewind`) does NOT restore in-process teammates. After resume, lead must re-spawn fresh teammates (state on disk survives).
- One team per session. If a previous /solve team is still alive, ask user before TeamDelete.
- Token cost scales linearly with active teammates. The 5+6 = 11 teammates during Six Hats burst is intentional but expensive.
- No nested teams: teammates cannot spawn their own teammates. Only the lead spawns hat-* during Phase 3.

---

## Helpers

- `bin/parallel-research <queries.json> <out-dir>` — Parallel.ai fan-out, falls back to WebSearch instructions
- `bin/parallel-setup` — interactive paste-friendly Parallel.ai key setup
- `bin/state-rw read|write|init <run-dir> <key> [value]` — JSON state for resume safety
- `bin/cost-track <usd>` — increments `$SOLVE_COST_FILE`
- `bin/render-report` — assembles final markdown
- `install` — one-shot installer (run once per machine)

---
> Source: [bajpainaman/solve](https://github.com/bajpainaman/solve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
