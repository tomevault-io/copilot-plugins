## pipeline-orchestrator

> Staged execution pipeline with plan→prd→exec→verify→fix loop, dual reasoning (think/act), parallel sub-agent spawning, and resumable state. Inspired by OMC and Chat2Graph. Use for any complex multi-step task that benefits from structured decomposition, verification, and automatic retry.


# Pipeline Orchestrator

A structured execution engine for complex, multi-step tasks. Decomposes work into a dependency graph, writes acceptance criteria, executes with parallel sub-agents, verifies results, and auto-retries failures — all with resumable state.

## When to Use This Skill

**Trigger phrases:**
- "Run pipeline: ..."
- "Pipeline status"
- "Resume pipeline ..."
- "Cancel pipeline ..."

**Good fit when:**
- Task has 3+ distinct sub-tasks
- Sub-tasks have dependencies (some must finish before others start)
- Quality matters — you want verification, not just execution
- Task is complex enough that a single-shot attempt risks missing pieces
- You want parallel execution to save time

**Skip this when:**
- Simple single-step task (just do it)
- Pure conversation / Q&A
- Task is exploratory with no clear deliverables

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    PIPELINE                          │
│                                                     │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌────────┐      │
│  │ PLAN │──▶│ PRD  │──▶│ EXEC │──▶│ VERIFY │      │
│  └──────┘   └──────┘   └──────┘   └────────┘      │
│                                        │            │
│                                   pass │ fail       │
│                                   ▼    ▼            │
│                                 DONE  ┌─────┐      │
│                                       │ FIX │──┐   │
│                                       └─────┘  │   │
│                                         ▲      │   │
│                                         └──────┘   │
│                                      (max 3 loops) │
│                                                     │
│  State: ~/.openclaw/workspace/state/pipeline-{id}/  │
└─────────────────────────────────────────────────────┘
```

Each stage uses the **Dual Reasoning** pattern: Think first (analyze, plan, assess risk), then Act (execute the plan, no improvisation).

---

## Pipeline Stages

### Stage 1: PLAN

**Purpose:** Decompose the task into a directed acyclic graph (DAG) of sub-tasks.

**Think Phase:**
Before creating the plan, reason through:
- What are ALL the discrete pieces of work?
- What depends on what? (draw the DAG mentally)
- Which tasks can run in parallel?
- What's the complexity of each? (simple: <5 min, medium: 5-20 min, complex: 20+ min)
- What could go wrong at each step?
- Are there any implicit dependencies the user didn't mention?

**Act Phase:**
Write `state/pipeline-{id}/plan.json` with this structure:

```json
{
  "id": "pipeline-abc123",
  "task": "Original task description from user",
  "created": "2026-04-04T04:00:00Z",
  "tasks": [
    {
      "id": "t1",
      "name": "Short descriptive name",
      "description": "What needs to be done",
      "dependencies": [],
      "complexity": "simple|medium|complex",
      "parallelGroup": 1,
      "contextFiles": ["path/to/relevant/file.js"]
    },
    {
      "id": "t2",
      "name": "Another task",
      "description": "Depends on t1 output",
      "dependencies": ["t1"],
      "complexity": "medium",
      "parallelGroup": 2,
      "contextFiles": []
    }
  ],
  "parallelGroups": {
    "1": ["t1", "t3"],
    "2": ["t2", "t4"],
    "3": ["t5"]
  },
  "estimatedDuration": "~15 minutes",
  "risks": ["Risk 1", "Risk 2"]
}
```

**Key rules:**
- `parallelGroup` numbers define execution waves. Group 1 runs first, then group 2 after group 1 completes, etc.
- Tasks in the same `parallelGroup` MUST have no dependencies on each other.
- `dependencies` lists task IDs that must complete before this task starts.
- `contextFiles` — only include files the sub-agent actually needs. Less is more.

**Advisory model:** Sonnet (text generation, no code execution needed).

---

### Stage 2: PRD (Product Requirements Document)

**Purpose:** Define explicit, testable acceptance criteria for every sub-task.

**Think Phase:**
For each task in the plan, reason through:
- What does "done" look like, specifically?
- How will we verify this programmatically vs. manually?
- What edge cases should the acceptance criteria cover?
- Are the criteria tight enough to catch a bad implementation?

**Act Phase:**
Write `state/pipeline-{id}/prd.json`:

```json
{
  "pipelineId": "pipeline-abc123",
  "requirements": [
    {
      "taskId": "t1",
      "taskName": "Short descriptive name",
      "acceptanceCriteria": [
        "File X exists at path Y",
        "Function Z returns correct output for inputs [a, b, c]",
        "No lint errors in modified files"
      ],
      "verificationMethod": "automated|manual|hybrid",
      "verificationCommands": [
        "test -f path/to/file.js && echo PASS || echo FAIL",
        "node -e \"require('./module').test()\" 2>&1"
      ],
      "verificationChecklist": [
        "Output matches expected format",
        "No regressions in existing functionality"
      ]
    }
  ]
}
```

**Key rules:**
- Every acceptance criterion must be binary: pass or fail. No "looks good enough."
- `verificationCommands` — shell commands that return PASS/FAIL or exit 0/1. Used in VERIFY stage.
- `verificationChecklist` — for things that need agent judgment (manual review).
- If you can't write a verification command, write a clear checklist item instead.

**Advisory model:** Sonnet (text generation, analytical).

---

### Stage 3: EXEC (Execution)

**Purpose:** Execute sub-tasks using parallel sub-agents where possible.

**Think Phase:**
Before spawning agents, reason through:
- Which parallel group are we executing?
- Do we have all prerequisite outputs from prior groups?
- What's the minimal context each sub-agent needs?
- Are there any shared resources that could conflict (same file edited by two agents)?
- Should any tasks be merged to avoid conflicts?

**Act Phase:**

**Parallel execution via `sessions_spawn`:**

For each parallel group (in order), spawn sub-agents for all tasks in that group. Maximum 4 concurrent sub-agents.

Each sub-agent prompt MUST include:
1. **Task description** (from plan.json)
2. **Acceptance criteria** (from prd.json)
3. **Relevant context** (read contextFiles and include content, not just paths)
4. **Output expectations** (what files to create/modify, what to report back)
5. **Constraints** (don't modify files outside scope, don't install new dependencies without noting it)

**Sub-agent prompt template:**

```
You are executing a sub-task in a pipeline.

## Your Task
{task.description}

## Acceptance Criteria (you MUST meet ALL of these)
{for each criterion in prd.requirements[taskId].acceptanceCriteria}
- [ ] {criterion}

## Context
{content of contextFiles}

## Output
When done, report:
1. What you did (brief)
2. Files created/modified (list)
3. Self-assessment against each acceptance criterion (pass/fail per criterion)
4. Any issues or concerns

Do NOT modify files outside your task scope.
Do NOT install packages without documenting them.
Stay focused. Complete the task. Report results.
```

**Progress tracking:**
Initialize and update `state/pipeline-{id}/progress.json`:

```json
{
  "pipelineId": "pipeline-abc123",
  "currentGroup": 1,
  "tasks": {
    "t1": {
      "status": "pending|running|completed|failed",
      "startedAt": null,
      "completedAt": null,
      "agentSessionId": null,
      "result": null,
      "filesModified": [],
      "selfAssessment": {}
    }
  }
}
```

Update `progress.json` as each sub-agent completes:
- When spawning: set `status: "running"`, record `startedAt` and `agentSessionId`
- When complete: set `status: "completed"`, record `completedAt`, `result`, `filesModified`, `selfAssessment`
- When failed: set `status: "failed"`, record error details in `result`

**Execution flow:**
```
for each parallelGroup in order:
  spawn sub-agents for all tasks in group (max 4 concurrent)
  yield and wait for all to complete
  update progress.json
  if any failed: continue (will be caught in VERIFY)
  proceed to next group
```

**If a parallel group has >4 tasks:** batch them in sets of 4. Wait for a batch to complete before spawning the next.

**Advisory model:** Opus (actual implementation work — sub-agents need the best model).

---

### Stage 4: VERIFY

**Purpose:** Check every sub-task against its acceptance criteria.

**Think Phase:**
Before verifying, reason through:
- Which tasks reported success vs. failure in self-assessment?
- Are there any suspicious "pass" claims that need deeper checking?
- Do the automated verification commands still make sense given what was actually built?
- Are there cross-task integration concerns (task A and B both pass individually but break together)?

**Act Phase:**

For each task, run verification:

1. **Automated checks:** Execute each `verificationCommand` from prd.json via `exec`. Record stdout/stderr and exit code.
2. **Checklist review:** For each `verificationChecklist` item, read the relevant files and assess pass/fail.
3. **Cross-task integration:** If multiple tasks modified related files, check for conflicts or inconsistencies.

Write `state/pipeline-{id}/verify.json`:

```json
{
  "pipelineId": "pipeline-abc123",
  "verifiedAt": "2026-04-04T04:15:00Z",
  "attempt": 1,
  "results": [
    {
      "taskId": "t1",
      "verdict": "pass|fail|partial",
      "criteria": [
        {
          "criterion": "File X exists at path Y",
          "result": "pass",
          "evidence": "File exists, 142 lines"
        },
        {
          "criterion": "Function Z returns correct output",
          "result": "fail",
          "evidence": "Returns undefined instead of expected array",
          "failureDetails": "Function exists but doesn't handle empty input case"
        }
      ],
      "automatedResults": [
        {
          "command": "test -f path/to/file.js",
          "exitCode": 0,
          "output": "",
          "result": "pass"
        }
      ]
    }
  ],
  "summary": {
    "total": 5,
    "passed": 3,
    "failed": 1,
    "partial": 1,
    "allPassed": false
  }
}
```

**Verdict logic:**
- `pass` — ALL acceptance criteria met
- `fail` — one or more criteria clearly not met
- `partial` — criteria ambiguous or partially met (treat as fail for retry purposes)

**If all passed:** Pipeline complete. Update meta.json status to `"completed"`. Report to user.
**If any failed/partial:** Proceed to FIX stage.

**Advisory model:** Sonnet (comparison/checking, not implementation).

---

### Stage 5: FIX (Bounded Retry Loop)

**Purpose:** Re-execute only failed sub-tasks with additional context about what went wrong.

**Think Phase:**
Before fixing, reason through:
- What exactly failed and why?
- Is the failure in the implementation, or in the acceptance criteria (bad spec)?
- Does the fix require changes to upstream tasks?
- Have we tried this fix before? (check prior attempts in verify.json)
- Is this fixable by an agent, or does it need human intervention?

**Act Phase:**

For each failed/partial task, spawn a fix sub-agent with an enhanced prompt:

```
You are FIXING a failed sub-task (attempt {N} of 3).

## Original Task
{task.description}

## What Was Done Previously
{previous result from progress.json}

## What Failed
{failure details from verify.json}

## Acceptance Criteria (you MUST meet ALL)
{criteria list}

## Fix Instructions
Focus specifically on the failure points. Do not redo work that already passes.
The following criteria FAILED:
{list of failed criteria with evidence}

Fix these issues. Report your changes.
```

**Retry logic:**
```
for attempt in 1..3:
  fix failed tasks
  run VERIFY again
  if all pass: done
  if still failing after attempt 3:
    mark remaining failures as "needs-human"
    report to user with details
```

Update `meta.json` with attempt count after each loop.

**Advisory model:** Opus (needs deep understanding to fix failures).

---

## Dual Reasoning Pattern

Every stage uses Think → Act. This is implemented as a prompting discipline, not separate model calls.

**Think prompt prefix (prepend to every stage):**

```
THINK PHASE — Do not take any action yet.

Analyze the following before proceeding:
1. SITUATION: What is the current state? What do we know?
2. RISKS: What could go wrong in this stage?
3. DEPENDENCIES: What does this stage depend on? What depends on it?
4. EDGE CASES: What non-obvious scenarios should we handle?
5. PRIOR FAILURES: Have previous attempts failed? Why? (check verify.json)
6. PLAN: What specific steps will we take, in what order?

Output your reasoning in the format above, then proceed to ACT.
```

**Act prompt suffix (append after think output):**

```
ACT PHASE — Execute the plan from your THINK phase.

Rules:
- Follow your plan. No improvisation.
- If you discover something unexpected, note it but stay on plan.
- Report results in structured format.
- If blocked, explain why rather than guessing.
```

See `references/dual-reasoning.md` for full pattern details.

---

## State Management

All state lives in `~/.openclaw/workspace/state/pipeline-{id}/`.

**Files:**
| File | Created at | Updated at | Purpose |
|------|-----------|-----------|---------|
| `meta.json` | PLAN start | Every stage | Pipeline metadata, current stage, status |
| `plan.json` | PLAN end | Never | Task decomposition & DAG |
| `prd.json` | PRD end | Never | Acceptance criteria |
| `progress.json` | EXEC start | During EXEC, FIX | Execution status per task |
| `verify.json` | VERIFY end | Each VERIFY run | Verification results |

**meta.json schema:**
```json
{
  "id": "pipeline-abc123",
  "task": "Original task description",
  "status": "planning|prd|executing|verifying|fixing|completed|failed|cancelled|needs-human",
  "currentStage": "PLAN|PRD|EXEC|VERIFY|FIX",
  "created": "ISO timestamp",
  "updated": "ISO timestamp",
  "completedAt": null,
  "fixAttempts": 0,
  "maxFixAttempts": 3,
  "taskCount": 5,
  "passedCount": 0,
  "failedCount": 0
}
```

**Resumability:**
When the agent starts and a pipeline command is given (or on "Resume pipeline X"):
1. Read `meta.json` to find current stage and status.
2. If status is `"executing"` — check `progress.json` for incomplete tasks. Re-spawn only those.
3. If status is `"verifying"` — re-run verification from scratch.
4. If status is `"fixing"` — check attempt count, continue fix loop.
5. If status is `"completed"` or `"cancelled"` — inform user, no action needed.

See `references/state-schema.md` for full JSON schemas.

---

## Pipeline ID Generation

Generate a short, unique ID for each pipeline:

```bash
date +%s | md5sum | head -c 8
```

Or in the agent: use the first 8 characters of a hash of the task description + timestamp. The ID must be filesystem-safe (alphanumeric + hyphens only).

---

## Smart Model Routing (Advisory)

These are recommendations. The orchestrating agent should request the appropriate model when spawning sub-agents, if the platform supports it.

| Stage | Recommended Model | Rationale |
|-------|------------------|-----------|
| PLAN | Sonnet | Text decomposition, no code needed |
| PRD | Sonnet | Analytical writing, criteria definition |
| EXEC | Opus | Actual implementation, code quality matters |
| VERIFY | Sonnet | Comparison against criteria, lightweight |
| FIX | Opus | Debugging requires deep understanding |

In OpenClaw, model can be suggested via sub-agent instructions ("Use careful, thorough reasoning for this implementation task").

---

## Invocation Reference

### Start a new pipeline
User says: `"Run pipeline: Build a REST API with auth, CRUD for users, and deploy script"`

Agent action:
1. Generate pipeline ID
2. Create state directory: `mkdir -p ~/.openclaw/workspace/state/pipeline-{id}`
3. Write `meta.json` with status `"planning"`
4. Execute PLAN stage
5. Continue through stages automatically

### Check status
User says: `"Pipeline status"` or `"Pipeline status abc123"`

Agent action:
1. If no ID given, find most recent pipeline in `state/` directory
2. Read `meta.json` and `progress.json`
3. Report: stage, status per task, pass/fail counts, elapsed time

Quick check via script:
```bash
bash ~/.openclaw/workspace/skills/pipeline-orchestrator/scripts/pipeline-status.sh [pipeline-id]
```

### Resume a pipeline
User says: `"Resume pipeline abc123"`

Agent action:
1. Read `state/pipeline-abc123/meta.json`
2. Determine last incomplete stage
3. Resume from that stage (see Resumability section above)

### Cancel a pipeline
User says: `"Cancel pipeline abc123"`

Agent action:
1. Update `meta.json`: set status to `"cancelled"`
2. Do NOT delete state files (useful for post-mortem)
3. Confirm cancellation to user

---

## Execution Walkthrough (Example)

Task: "Build a CLI tool that converts CSV to JSON with validation"

**PLAN output:**
```
t1: Parse CLI arguments (simple, group 1)
t2: CSV reader module (medium, group 1)  
t3: JSON writer module (simple, group 1)
t4: Validation logic (medium, group 2, depends: t2)
t5: Integration + main entry point (medium, group 3, depends: t1,t2,t3,t4)
t6: Tests (medium, group 4, depends: t5)
```

**Parallel groups:**
- Group 1: t1, t2, t3 (spawn 3 sub-agents simultaneously)
- Group 2: t4 (depends on t2 — wait for group 1)
- Group 3: t5 (depends on all prior)
- Group 4: t6 (depends on t5)

**EXEC:** 3 sub-agents run in parallel for group 1, then sequential for groups 2-4.

**VERIFY:** Run test commands, check file existence, validate output format.

**FIX:** If t4 validation logic fails edge case → re-spawn only t4 with failure context. Re-verify. Up to 3 attempts.

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Sub-agent crashes / no response | Mark task as `failed`, include in FIX stage |
| State file corrupted | Re-create from available state, log warning |
| All 3 fix attempts exhausted | Set status `needs-human`, report details to user |
| User cancels mid-execution | Set status `cancelled`, sub-agents may still complete (orphaned work is ok) |
| Circular dependency detected in plan | Abort PLAN stage, report error, ask user to clarify |
| >20 sub-tasks in plan | Warn user about complexity, suggest breaking into multiple pipelines |

---

## File References

- **Stage details:** `references/pipeline-stages.md`
- **Dual reasoning pattern:** `references/dual-reasoning.md`
- **State file schemas:** `references/state-schema.md`
- **Status script:** `scripts/pipeline-status.sh`

---
> Source: [Cat-tj/pipeline-orchestrator](https://github.com/Cat-tj/pipeline-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
