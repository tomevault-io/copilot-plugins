## dobby

> Summon Dobby to work on substantial tasks autonomously in the background. Use when the user wants to delegate a project (research paper, build an app, design a system) to an autonomous agent that works while they do other things. Triggers on "dobby", "summon", "cast", "delegate project", "run in background", "work on this while I".

# Dobby

Dobby is a free elf. Give Dobby a task and Dobby works on it autonomously in the background while you do other things. Each task gets its own folder, CLAUDE.md, convergence loop, and budget cap.

## Usage

```bash
/dobby <request>                     # Give Dobby a task
/dobby --role <slug> <request>       # Give Dobby a task with a specific roster role
/dobby --converge <request>          # Run with convergence loop (build → evaluate → fix)
/dobby status                        # Show all tasks and progress
/dobby progress <task-name>          # Live progress bar for a specific task
/dobby notify                        # Show notification config status
/dobby stop <task-name>              # Stop a running task
/dobby deliver <task-name>           # Read output and present results
/dobby roster                        # Show project agent roster (from .dobby/roster-config.md)
/dobby roster init                   # Create a starter .dobby/roster-config.md
/dobby tell <task-name> "message"    # Send async message to a running task (Tier 2 inbox)
/dobby tell <task-name> STOP         # Signal agent to pause at next Tier 2 checkpoint (escalate to Tier 3)
```

## First Run: Bootstrap `.dobby/`

If `.dobby/` does not exist in the current working directory, create it:

```bash
mkdir -p .dobby/records
```

Create empty roster at `.dobby/records/roster.md`:

```markdown
| Task | Folder | Status | Budget | Spent | Started | Last Update |
|---|---|---|---|---|---|---|
```

After bootstrap, proceed to routing.

## Routing Protocol

Not everything needs a background agent. Classify first, route second:

| Classification | Route | When |
|---|---|---|
| **Quick** (< 30 sec) | Do it yourself | "What time is it?", "Read this file" |
| **Task** (< 5 min) | Task tool (subagent) | Code review, quick analysis, draft email |
| **Project** (hours/days) | Dobby works on it in tmux | Research paper, build an app, design a system |
| **Multi-domain** | Multiple tasks in parallel | "Build an app AND write docs" |

**Default to the lightest route.** For Quick and Task: do the work directly (or via Task tool).

### Roster-Aware Routing (for Project classification)

After classifying as Project, check for a roster config:

```bash
ROSTER_CONFIG=".dobby/roster-config.md"
```

If `$ROSTER_CONFIG` exists:
1. If the user passed `--role <slug>`, use that role directly
2. Otherwise, score each role's keywords against the request text; pick the top match
3. Extract `MODEL`, `BUDGET`, and `DESCRIPTION` from the matched role row
4. Tell the user: `Using role: {Role} ({Model}, {Budget})`

If `$ROSTER_CONFIG` does not exist or no role matches, fall back to defaults:
- Model: `claude-sonnet-4-6`
- Budget: ask user or default `$5`

**Budget normalization:** Always strip the leading `$` from budget values before use in shell arithmetic or `--max-budget-usd`:
```bash
BUDGET="${BUDGET#\$}"
```

### Multi-Domain Orchestration (for Multi-domain classification)

When a request spans multiple domains ("build an app AND write docs", "implement the API, test it, and write a blog post"), decompose and launch a team:

**Step 1: Decompose**
Break the request into 2-5 independent subtasks. Each subtask should be:
- Self-contained (can run without the others)
- Mappable to a roster role (or use defaults)
- Named with a short slug

Example: "build a REST API and write docs" →
- `build-api` (keywords: build, implement → Builder role)
- `write-docs` (keywords: write, document → Writer role)

**Step 2: Match roles**
For each subtask, run the same Roster-Aware Routing:
- If roster-config.md exists, keyword-match to pick role
- Extract MODEL, BUDGET, DESCRIPTION per subtask
- If no roster, all subtasks use defaults (claude-sonnet-4-6, $5)

**Step 3: Scaffold all**
For each subtask:
```bash
SUBTASK="<slug>"
TASK_DIR=".dobby/${SUBTASK}"
mkdir -p "${TASK_DIR}/records" "${TASK_DIR}/output"
```

Write each CLAUDE.md with:
1. Role injection (if matched)
2. Mission (the subtask, not the full request)
3. **Team context block:**
   ```
   ## Team Context
   You are part of a team working on: "{original request}"
   Your specific task: "{subtask description}"
   Other team members and their output directories:
   - {role}: {subtask} → .dobby/{slug}/output/
   You may read other team members' output/ directories for coordination.
   ```
4. HITL protocol, state management, constraints (same as single-agent)

**Step 4: Set up ALL listeners**
For each subtask, start completion + question listeners (same pattern as single-agent, but repeated for each). Use unique signal names: `dobby_${SUBTASK}_done`, `dobby_${SUBTASK}_question`.

**Step 5: Launch ALL agents**
For each subtask, launch in its own tmux session: `dobby-${SUBTASK}`

Use the same launch command as single-agent Step 4, but with each subtask's TASK_DIR, MODEL, and BUDGET:
```bash
tmux new-session -d -s "dobby-${SUBTASK}"
tmux send-keys -t "dobby-${SUBTASK}" "cd $(pwd)/.dobby/${SUBTASK} && for attempt in 1 2 3; do claude -p --model ${MODEL} --dangerously-skip-permissions --max-budget-usd ${BUDGET} --output-format json ${MCP_FLAG} 'You are an autonomous agent. Read your CLAUDE.md. Do the work. Write all deliverables to output/. Update records/TODO.md with progress. If you need human help, follow the asking-for-help protocol in your CLAUDE.md. When complete, print COMPLETE.' 2>&1 | tee /tmp/dobby_${SUBTASK}_output.txt && break || sleep 15; done ; tmux wait-for -S dobby_${SUBTASK}_done" Enter
```

**Step 6: Update roster**
Add a row per subtask to `.dobby/records/roster.md`.

**Step 7: Report**
```
Dobby team assembled for: {original request}
  - {subtask-1} ({role}, ${budget}) → .dobby/{slug-1}/output/
  - {subtask-2} ({role}, ${budget}) → .dobby/{slug-2}/output/
```

### Team Completion

When ALL completion listeners fire, first cancel any pending question listeners so they don't block forever:
```bash
for SUBTASK in ${SUBTASK_LIST//,/ }; do tmux wait-for -S dobby_${SUBTASK}_question 2>/dev/null || true; done
```
Then:
1. Parse cost from each agent's `/tmp/dobby_${SUBTASK}_output.txt`
2. Read deliverables from each `.dobby/${SUBTASK}/output/`
3. Update roster: all subtasks → Complete
4. Report:
   ```
   Dobby team finished: {original request}
   Total cost: ${sum} across {N} agents
   Deliverables:
     - {subtask-1}: .dobby/{slug-1}/output/
     - {subtask-2}: .dobby/{slug-2}/output/
   ```
4b. **Notify team done (if configured):**
    ```bash
    SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
    uv run "${SKILL_DIR}/notify/webhook.py" team_done --tasks "${SUBTASK_LIST}" --total-cost "${TOTAL_COST}" --request "${ORIGINAL_REQUEST}" 2>/dev/null || true
    ```
    where SUBTASK_LIST is a comma-separated list of subtask slugs.

## Giving Dobby a Task

### Step 1: Classify the Request

Determine from the user's request:
- **Task name**: short slug (e.g., `blog-post`, `api`, `docs`, `analysis`)
- **Budget**: `$5` for small, `$20` for medium, `$50` for large tasks (or from matched role)
- **Success criteria**: what should be delivered to `output/`

If classification is **Multi-domain**: follow the Multi-Domain Orchestration protocol above instead of Steps 2-6 below. Steps 2-6 are for single-agent (Project) classification only.

**Convergence mode detection:** Enable convergence loop if ANY of:
- User passed `--converge` flag
- Request contains quality signals: "iterate", "converge", "review loop", "quality gate", "keep improving", "until it's good"
- Budget is $20+ (substantial tasks benefit from iteration)

If convergence is enabled, follow Steps 2-6 below for initial launch, then the Convergence Loop protocol kicks in after the first completion.

If `.dobby/roster-config.md` exists, run Roster-Aware Routing now and set `MODEL` and `BUDGET` from the matched role. Otherwise use defaults.

If the request is ambiguous, ask the user ONE question to clarify. Don't over-ask.

### Step 2: Scaffold the Task Folder

```bash
TASK_NAME="<name>"
TASK_DIR=".dobby/${TASK_NAME}"
mkdir -p "${TASK_DIR}/records" "${TASK_DIR}/output"
```

Write a `${TASK_DIR}/CLAUDE.md` tailored to the task. Include:

1. **Role** (if matched from roster): prepend a `# Your Role: {Role}` section with the role's description followed by a `---` separator. This persona section comes FIRST.
2. **Mission**: what Dobby should deliver (from user's request)
3. **Success criteria**: what "done" looks like
4. **Budget**: the cap
5. **Human-in-the-loop protocol**: (copy this exactly, replacing TASKNAME with the actual task name slug)
   ```
   ## Human-in-the-loop Protocol

   **Tier selection:**
   - Need credentials, QR scan, physical action, deploy to prod, or modify files outside output/? -> Tier 3
   - At a phase transition (planning done, core work done, pre-delivery)? -> Tier 2
   - Everything else -> Tier 1 (DEFAULT: log and continue)

   ### Tier 1 (DEFAULT — non-blocking)
   For all routine decisions, make your best judgment and log it to `records/DECISIONS.md`:
   ```
   | <Decision> | <Rationale> | <Date> |
   ```
   No blocking. Continue immediately.

   ### Tier 2 (checkpoint — phase transitions)
   At major phase transitions (planning complete, core work done, before final delivery):
   1. Write a brief summary to `records/CHECKPOINT.md`
   2. Signal: `tmux wait-for -S dobby_TASKNAME_checkpoint`
   3. Wait: `sleep 60`
   4. If `records/INBOX.md` has content, read and incorporate it, then delete it
   5. If `records/STOP` exists:
      - Write to `records/QUESTION.md`: "Human requested a pause via STOP file. Awaiting instructions."
      - Escalate: follow Tier 3 steps 2–5 (signal, wait, read answer, delete files)
   6. Continue working

   ### Tier 3 (hard block — rare)
   ONLY for: credentials needed, QR codes to scan, physical actions required, deploying to production infrastructure, modifying files outside output/.
   1. Write your question to `records/QUESTION.md` — explain what you need and why
   2. Run: `tmux wait-for -S dobby_TASKNAME_question`
   3. Run: `timeout 1800 tmux wait-for dobby_TASKNAME_answer || echo "Timed out after 30min, proceeding with best judgment"`
   4. Read the answer from `records/ANSWER.md` (if it exists)
   5. Delete both files and continue working
   ```
5. **State management**: "Track progress in `records/TODO.md`. Write deliverables to `output/`."
6. **Constraints**: from user's request or "None specified"

**If convergence mode is enabled**, additionally:
- Create `records/version_registry.md` with headers: `| Version | Date | Strategy | Score | Key Change |`
- Create `records/loop_state.json`: `{"iteration": 0, "max_iterations": 5, "scores": [], "status": "running"}`
- Determine task type: `research` (paper, analysis) or `engineering` (code, system) or `general`
- If `research` or `engineering` overlay exists at `${SKILL_DIR}/templates/overlays/`, append it to the task's CLAUDE.md
- Write `quality_rubric.md` in `${TASK_DIR}` with evaluation criteria extracted from the user's request and success criteria
- Partition the budget: 70% production, 20% quality, 10% reserve. Set `ITER_BUDGET = BUDGET * 0.14` and `QUALITY_BUDGET = BUDGET * 0.04` (supports ~5 iterations)

### Step 3: Set Up Listeners BEFORE Launch

**Pre-flight check:** Verify tmux is available before any launch:
```bash
command -v tmux >/dev/null 2>&1 || { echo "Dobby requires tmux. Install: sudo apt install tmux (or brew install tmux)"; exit 1; }
```

**IMPORTANT:** Start listeners before launching the agent to avoid signal races. If the agent sends a signal before the listener is ready, tmux drops it silently.

Run TWO background listeners with `run_in_background: true`:

**Completion listener (with timeout):**
```bash
BUDGET_NUM="${BUDGET#\$}"
timeout $((BUDGET_NUM * 720)) tmux wait-for dobby_${TASK_NAME}_done
```
The timeout is ~12 minutes per dollar of budget. If the timeout fires, check `tmux list-sessions` for the session. Tell the user: "Dobby timed out on {task-name}. Partial output may be in .dobby/{task-name}/output/." Update roster status to "Timed Out".

**Question listener (persistent loop):**
```bash
while true; do tmux wait-for dobby_${TASK_NAME}_question 2>/dev/null || break; echo "QUESTION_RECEIVED"; break; done
```

**Checkpoint listener (non-blocking, optional):**
```bash
while true; do
  tmux wait-for dobby_${TASK_NAME}_checkpoint 2>/dev/null || { sleep 2; continue; }
  SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
  SUMMARY=$(cat ".dobby/${TASK_NAME}/records/CHECKPOINT.md" 2>/dev/null | head -3 || echo "Checkpoint reached")
  uv run "${SKILL_DIR}/notify/webhook.py" checkpoint --task "${TASK_NAME}" --summary "${SUMMARY}" 2>/dev/null || true
  # Also dispatch the latest decision logged in DECISIONS.md (non-blocking, fire-and-forget)
  LATEST_DECISION=$(tail -1 ".dobby/${TASK_NAME}/records/DECISIONS.md" 2>/dev/null | tr -d '|' | xargs || true)
  if [ -n "${LATEST_DECISION}" ]; then
    uv run "${SKILL_DIR}/notify/webhook.py" decision --task "${TASK_NAME}" --decision "${LATEST_DECISION}" --rationale "see records/DECISIONS.md" 2>/dev/null || true
  fi
  # Non-blocking: re-arm loop continues — agent is not interrupted by this listener
done
```

When the completion listener fires → Dobby is done. First, cancel the pending question listener so it does not block forever (the agent may have finished without asking any questions):
```bash
tmux wait-for -S dobby_${TASK_NAME}_question 2>/dev/null || true
```
Then proceed to Task Completion below.

When the question listener fires:
1. Read `.dobby/${TASK_NAME}/records/QUESTION.md`
1b. **Notify (if configured):**
    ```bash
    SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
    uv run "${SKILL_DIR}/notify/webhook.py" question --task "${TASK_NAME}" --message "$(cat .dobby/${TASK_NAME}/records/QUESTION.md)" 2>/dev/null || true
    ```
1c. **Start relay (if Level 2 bot configured):**
    ```bash
    uv run "${SKILL_DIR}/notify/relay.py" listen --task "${TASK_NAME}" --timeout 300 &
    RELAY_PID=$!
    ```
2. Present the question to the user (use AskUserQuestion or just show it)
3. Check if `.dobby/${TASK_NAME}/records/ANSWER.md` already exists (bot may have written it):
   - If yes: read it, skip writing, kill relay (`kill $RELAY_PID 2>/dev/null`)
   - If no: write the user's answer to ANSWER.md
4. Signal back: run `tmux wait-for -S dobby_${TASK_NAME}_answer`
5. Kill relay if still running: `kill $RELAY_PID 2>/dev/null`
6. Start a NEW question listener (the agent may ask again)

### Step 4: Launch in tmux

Before launching, check for MCP configs to pass through:
```bash
MCP_FLAG=""
if [ -f .mcp.json ]; then MCP_FLAG="--mcp-config .mcp.json"; fi
```

```bash
TASK_NAME="<name>"
TASK_DIR="$(pwd)/.dobby/${TASK_NAME}"
BUDGET="<budget>"             # from roster role, or determined in Step 1
MODEL="claude-sonnet-4-6"     # from roster role, or default

tmux new-session -d -s "dobby-${TASK_NAME}"
tmux send-keys -t "dobby-${TASK_NAME}" "cd ${TASK_DIR} && for attempt in 1 2 3; do claude -p --model ${MODEL} --dangerously-skip-permissions --max-budget-usd ${BUDGET} --output-format json ${MCP_FLAG} 'You are an autonomous agent. Read your CLAUDE.md. Do the work. Write all deliverables to output/. Update records/TODO.md with progress. If you need human help, follow the asking-for-help protocol in your CLAUDE.md. When complete, print COMPLETE.' 2>&1 | tee /tmp/dobby_${TASK_NAME}_output.txt && break || sleep 15; done ; tmux wait-for -S dobby_${TASK_NAME}_done" Enter
```

### Step 5: Update Roster

Add a row to `.dobby/records/roster.md`:

```
| {TASK_NAME} | {TASK_NAME} | Running | ${BUDGET} | $0.00 | {YYYY-MM-DD} | {YYYY-MM-DD} |
```

### Step 6: Report

Tell the user:
```
Dobby is on it: {task-name}
Budget: ${BUDGET} | Output: .dobby/{task-name}/output/
```

That's it. Short and minimal. The user will be notified when the task completes.

## Task Completion

When the `tmux wait-for` background listener fires:

1. Parse `/tmp/dobby_${TASK_NAME}_output.txt` — find the last JSON line for cost data:
   ```json
   {"total_cost_usd": X, "num_turns": Y, "duration_ms": Z, "modelUsage": {...}}
   ```
2. Read `.dobby/${TASK_NAME}/output/` for deliverables
3. Update `.dobby/records/roster.md`: status → Complete, spent → actual cost
4. Tell the user:
   ```
   Dobby has finished: {task-name}
   Cost: ${X} | Deliverables: .dobby/{task-name}/output/
   ```
4b. **Notify (if configured):**
    ```bash
    SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
    uv run "${SKILL_DIR}/notify/webhook.py" completed --task "${TASK_NAME}" --cost "${COST}" --output-dir ".dobby/${TASK_NAME}/output/" 2>/dev/null || true
    ```
5. Offer to present the deliverables

## Convergence Loop Protocol

When convergence mode is enabled, the orchestrator does NOT fire-and-forget. After the first agent completes, the orchestrator runs an iterative produce→evaluate→decide loop. The orchestrator stays active for the duration of the loop.

### Signal Naming

Use versioned signal names to prevent collisions across iterations:
- Production: `dobby_${TASK_NAME}_prod_v${N}_done`
- Quality: `dobby_${TASK_NAME}_qual_v${N}_done`
- HITL: `dobby_${TASK_NAME}_prod_v${N}_question` / `_answer`

### The Loop

For each iteration N (starting at 1, up to max_iterations):

**A. Produce**

Write or update the production agent's instructions:
- Iteration 1: use the CLAUDE.md from Step 2 as-is
- Iteration N>1: append `records/EVAL_FEEDBACK.md` to CLAUDE.md with: "A quality review found these issues. Fix them. Write improved deliverables to output/."

Set up listeners BEFORE launch (same as Step 3, but with versioned signals):
```bash
# Completion listener (run_in_background: true)
timeout $((ITER_BUDGET * 720)) tmux wait-for dobby_${TASK_NAME}_prod_v${N}_done

# Question listener (run_in_background: true)
while tmux wait-for dobby_${TASK_NAME}_prod_v${N}_question 2>/dev/null; do echo "QUESTION_RECEIVED"; break; done
```

Launch production agent (session name includes iteration to prevent collisions):
```bash
tmux new-session -d -s "dobby-${TASK_NAME}-prod-v${N}"
tmux send-keys -t "dobby-${TASK_NAME}-prod-v${N}" "cd ${TASK_DIR} && claude -p --model ${MODEL} --dangerously-skip-permissions --max-budget-usd ${ITER_BUDGET} --output-format json ${MCP_FLAG} 'You are a production agent. Read your CLAUDE.md. Build or improve the deliverable. Write to output/. When complete, print COMPLETE.' 2>&1 | tee /tmp/dobby_${TASK_NAME}_prod_v${N}.txt ; tmux wait-for -S dobby_${TASK_NAME}_prod_v${N}_done" Enter
```

Wait for completion. Handle HITL questions as usual (read QUESTION.md, relay to user, write ANSWER.md, signal back). When the completion listener fires, cancel the question listener:
```bash
tmux wait-for -S dobby_${TASK_NAME}_prod_v${N}_question 2>/dev/null || true
```

**B. Evaluate (quality agent, isolated)**

Copy deliverables to an isolated temp directory so the quality agent cannot see prior scores, version history, or the vision:
```bash
QUAL_DIR=$(mktemp -d /tmp/dobby_qual_${TASK_NAME}_XXXX)
cp -r ${TASK_DIR}/output/* ${QUAL_DIR}/
cp ${TASK_DIR}/quality_rubric.md ${QUAL_DIR}/CLAUDE.md
```

Set up quality listener:
```bash
timeout $((QUALITY_BUDGET * 720)) tmux wait-for dobby_${TASK_NAME}_qual_v${N}_done
```

Launch quality agent in the isolated directory (session name includes iteration):
```bash
QUALITY_MODEL="claude-sonnet-4-6"  # or from roster "reviewer" role
tmux new-session -d -s "dobby-${TASK_NAME}-qual-v${N}"
tmux send-keys -t "dobby-${TASK_NAME}-qual-v${N}" "cd ${QUAL_DIR} && claude -p --model ${QUALITY_MODEL} --dangerously-skip-permissions --max-budget-usd ${QUALITY_BUDGET} --output-format json 'You are an independent quality evaluator. Read CLAUDE.md for the rubric. Evaluate the artifacts in this directory. Be honest and critical. Write your evaluation to EVAL_RESULT.json with format: {\"score\": <1-10>, \"dimensions\": {\"dim1\": <score>, ...}, \"issues\": [\"issue1\", ...], \"strengths\": [\"strength1\", ...]}. When complete, print COMPLETE.' 2>&1 | tee /tmp/dobby_${TASK_NAME}_qual_v${N}.txt ; tmux wait-for -S dobby_${TASK_NAME}_qual_v${N}_done" Enter
```

After quality completes, copy the evaluation back and clean up:
```bash
cp ${QUAL_DIR}/EVAL_RESULT.json ${TASK_DIR}/records/EVAL_RESULT_v${N}.json
rm -rf ${QUAL_DIR}
```

**C. Decide (orchestrator does this directly)**

Read `records/EVAL_RESULT_v${N}.json`. Parse score and issues.

Update `records/version_registry.md`:
```
| v${N} | ${DATE} | ${STRATEGY} | ${SCORE} | ${KEY_INSIGHT} |
```

Update `records/loop_state.json` with the new score.

**Cumulative budget check:** Before deciding to iterate, sum costs from all `/tmp/dobby_${TASK_NAME}_prod_v*.txt` and `qual_v*.txt` files. If cumulative > BUDGET * 0.9, force-ship or escalate to user.

**Notify convergence update (if configured):**
```bash
SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
uv run "${SKILL_DIR}/notify/webhook.py" convergence --task "${TASK_NAME}" --iteration "${N}" --score "${SCORE}" --decision "${DECISION}" 2>/dev/null || true
```

Decision gate:
- **Ship** if score >= 8 (or user's target) OR N == max_iterations → proceed to Task Completion
- **Reset** if score dropped >2 points below peak → copy peak version's output back, note "reset to v{peak}" in strategy
- **Escalate** if score plateaued for 3 iterations (last 3 scores within +/-0.5) → ask user: "Score plateaued at {score}. Continue, adjust strategy, or ship current best?"
- **Iterate** otherwise → write `records/EVAL_FEEDBACK.md` from EVAL_RESULT issues, increment N, go to step A

**D. Record**

After each decide step, update `records/loop_state.json` and write the orchestrator's current position so it can resume after context compaction:
```json
{
  "iteration": N,
  "max_iterations": 5,
  "scores": [6.2, 7.1, ...],
  "status": "waiting_for_quality_v3",
  "peak_version": 2,
  "peak_score": 7.1,
  "next_action": "launch_production_v4"
}
```

### Convergence Completion

When the loop finishes (ship decision):
1. Parse costs from ALL `/tmp/dobby_${TASK_NAME}_prod_v*.txt` and `qual_v*.txt` files
2. Sum total cost across all iterations
3. Read final deliverables from `output/`
4. Update roster: status → Complete, spent → total cost
5. Report:
   ```
   Dobby has finished: {task-name} (converged in {N} iterations)
   Score trajectory: v1={s1} → v2={s2} → ... → v{N}={sN}
   Total cost: ${total} ({N} production + {N} quality evaluations)
   Deliverables: .dobby/{task-name}/output/
   ```
6. Show the version registry summary

### Bottleneck Detection (checked after each Decide step, iteration >= 2)

| Condition | Action |
|---|---|
| Last 3 scores within +/-0.5 (plateau) | Launch advisory agent to diagnose binding constraint. Include `records/ADVISORY.md` in next EVAL_FEEDBACK. |
| Score >8 but output has issues | Upgrade quality model or add second quality agent with different rubric. |
| Advisory says "structural problem" 2x | Launch parallel variant strategies. Evaluate both. Pick winner. |
| Cost > 60% budget AND score < 50% target | Escalate to user: "Spent ${X} of ${BUDGET}, score is ${S}. Continue or stop?" |

### Session Resumption (after context compaction)

On session start or after compaction, if a convergence task exists:
1. Read `records/loop_state.json` — this is the ground truth
2. Check `tmux list-sessions` for active agents
3. Based on `next_action` in loop_state, resume the loop at the right step
4. If an agent is still running, re-establish the listener and wait

## Status Command

When the user runs `/dobby status`:
1. Read `.dobby/records/roster.md` for all tasks
2. Run `tmux list-sessions` to check which are still alive
3. For each active task, read `.dobby/{name}/records/TODO.md` for latest progress
4. For each active task, parse `/tmp/dobby_{name}_output.txt` for current cost
5. Display a summary:
   ```
   Task           Status    Budget   Spent    Progress
   research       Running   $20.00   $3.42    Phase 5: Inner loop, v3 in review
   api            Complete  $10.00   $7.80    Delivered: REST API with tests
   ```

## Progress Command

When the user runs `/dobby progress <task-name>`:

```bash
SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
uv run "${SKILL_DIR}/progress.py" "${TASK_NAME}"
```

This shows a live progress bar that refreshes every 5 seconds, tracking:
- Phase completion (Task configured → Records initialized → Work complete → Deliverable written)
- In-progress items from the task's TODO.md
- Output files with sizes
- Cost and duration (when available)

Pass `--once` for a single snapshot instead of live watching.

## Notifications (Slack/Discord Webhooks)

Dobby can push notifications to Slack and/or Discord when agents finish, ask questions, or update convergence scores. This is **optional** — notifications are silently disabled when no webhook is configured.

### Configuration

Set webhook URLs via environment variables or `.dobby/notify.conf`:

```bash
# Environment variables (one or both):
export DOBBY_SLACK_WEBHOOK="https://hooks.slack.com/services/T.../B.../xxx"
export DOBBY_DISCORD_WEBHOOK="https://discord.com/api/webhooks/123456/abcdef"
```

Or create `.dobby/notify.conf`:
```ini
[slack]
webhook = https://hooks.slack.com/services/T.../B.../xxx

[discord]
webhook = https://discord.com/api/webhooks/123456/abcdef

[notify]
events = all
```

### Notify Command

When the user runs `/dobby notify`:
1. Load config from env vars and `.dobby/notify.conf`
2. Show which platforms are configured and which events are enabled
3. If nothing configured, tell the user how to set it up

### Hook Points

All notification hooks are fire-and-forget (`2>/dev/null || true`). Webhook failure never blocks callers:
```bash
SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
uv run "${SKILL_DIR}/notify/webhook.py" <event_type> <args> 2>/dev/null || true
```
Events: `completed`, `question`, `convergence`, `team_done`, `command`, `decision`, `checkpoint`

### Command Interface (Remote Control)

When the relay daemon runs with `--enable-commands`, users can trigger tasks from Slack/Discord. Any message starting with `dobby ` (e.g., "dobby build a REST API") is validated against `authorized_users`, slugified, and launched in a tmux session (`dobby-cmd-{slug}`). Start the daemon:
```bash
SKILL_DIR="$(readlink -f ~/.claude/skills/dobby 2>/dev/null || echo ~/.claude/skills/dobby)"
cd /path/to/project && uv run "${SKILL_DIR}/notify/relay.py" daemon --enable-commands &
```
Security: Only `authorized_users` can trigger commands. Tasks execute locally with your permissions.

### Convergence + Multi-Domain Interaction

When both flags are active: subtasks run independently (no per-subtask convergence). After ALL complete, a single convergence pass runs on the combined output. This prevents N x M combinatorial explosion.

## Tell Command

When the user runs `/dobby tell <task-name> "message"`:

1. Resolve the task directory: `TASK_DIR=".dobby/${TASK_NAME}"`
2. If `${TASK_DIR}` does not exist, tell the user: "No task named {task-name} found."
3. Append the message to `${TASK_DIR}/records/INBOX.md`:
   ```bash
   echo "[$(date '+%Y-%m-%d %H:%M')] ${MESSAGE}" >> "${TASK_DIR}/records/INBOX.md"
   ```
4. Tell the user: "Message delivered to {task-name}. Dobby will pick it up at the next checkpoint."

**Special case: STOP signal**

If the message is exactly `STOP` (case-insensitive), instead of appending to INBOX.md:
1. Create `${TASK_DIR}/records/STOP`:
   ```bash
   touch "${TASK_DIR}/records/STOP"
   ```
2. Tell the user: "STOP signal sent to {task-name}. Dobby will pause and ask for instructions at the next checkpoint."

## Stop Command

When the user runs `/dobby stop <task-name>`:

1. `tmux send-keys -t dobby-{task-name} C-c`
2. Wait 3 seconds
3. `tmux kill-session -t dobby-{task-name}` if still running
4. Update roster: status → Stopped
5. Report whatever partial output exists in `.dobby/{name}/output/`

## Deliver Command

When the user runs `/dobby deliver <task-name>`:

1. Read all files in `.dobby/{task-name}/output/`
2. Present them to the user with a summary
3. If the task has a version registry, show the quality trajectory

## Roster Config

The **agent roster** allows a project to predefine named agent roles. Each role carries a model, budget cap, keyword hints for auto-matching, and a short description injected into the agent's CLAUDE.md persona.

### Format: `.dobby/roster-config.md`

```markdown
# Agent Roster

| Role | Slug | Model | Budget | Keywords | Description |
|---|---|---|---|---|---|
| Builder | builder | claude-sonnet-4-6 | $5 | implement,build,code,feature,fix | Expert software engineer who writes clean, tested code |
| Reviewer | reviewer | claude-haiku-4-5 | $2 | review,check,audit,inspect,scan | Meticulous reviewer who finds bugs and security issues |
| Researcher | researcher | claude-sonnet-4-6 | $10 | research,analyze,study,survey,investigate | Rigorous researcher who synthesizes information and cites sources |
| Writer | writer | claude-sonnet-4-6 | $8 | write,draft,document,blog,report,readme | Technical writer who produces clear, well-structured documents |
```

### Matching Logic

1. If user passed `--role <slug>`, use that slug directly (error if not found).
2. Otherwise, for each role, count how many of its keywords appear in the request text (case-insensitive). Pick the role with the highest count. If tied, use the first matching role in table order.
3. If no keywords match, fall back to defaults (model: `claude-sonnet-4-6`, budget: ask or `$5`).
4. Strip the leading `$` from the Budget column before passing to `--max-budget-usd`.

### Role Injection into CLAUDE.md

When a role is matched, prepend this block to the task's CLAUDE.md (before the Mission):

```markdown
# Your Role: {Role}

{Description}

Apply this expertise fully to the task below.

---
```

### Roster Commands

**`/dobby roster`** — display the current roster config in a readable table. If `.dobby/roster-config.md` doesn't exist, say so and suggest `init`.

**`/dobby roster init`** — create a starter `.dobby/roster-config.md` with four default roles (Builder, Reviewer, Researcher, Writer). Tell the user to edit it to fit their project.

## Operating Principles

1. **Route to the lightest handler** — don't summon Dobby for a 30-second task
2. **Parallel by default** — Dobby can work on multiple tasks simultaneously in separate tmux sessions
3. **Event-driven** — use tmux `wait-for` + `run_in_background`, never poll
4. **Deliver, don't report** — user wants the result, not status updates. Notify on completion.
5. **Cost-aware** — track every dollar, alert near budget cap
6. **Minimal by default** — short messages. Budget + output path is enough. Notifications are opt-in.

## tmux Conventions

- **DO NOT** use `tmux new-session -d -s name "command"` — skips `.bashrc`, API keys missing
- **DO:** `tmux new-session -d -s name` then `tmux send-keys -t name 'command' Enter`
- Use `;` between command and signal (fires even on failure). `&&` fires only on success.
- Session names prefixed with `dobby-` to avoid conflicts
- Use `uv run python` for scripts with dependencies

---
> Source: [zl190/dobby](https://github.com/zl190/dobby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
