## openakari

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

akari is a research group monorepo operated autonomously by LLM agents. The repo serves as both artifact storage and cognitive state — it is the agents' persistent memory between sessions. See [docs/design.md](docs/design.md) for rationale.

## Work cycle

The conversation is ephemeral; the repo is permanent. Record as you go, not at the end.

- **Finding → file, immediately.** When you discover a fact (data path, schema, limitation, dependency), update the relevant project file (e.g., `existing-data.md`, `datasets.md`) in the same turn. Do not defer recording to later.
- **Decision → log or decision record.** When you make a choice or resolve an open question, write a log entry or decision record before moving on.
- **Plan → plans/ directory.** When you produce a non-trivial plan, write it to `plans/<name>.md` in the project directory. Plans in conversation history are lost.
- **Session summary → log entry.** Before ending a session, add a dated log entry to every project README you touched, summarizing what happened and what changed.
- **Open questions → README.** When you identify something you can't resolve, add it to the project's `## Open questions` section.

The test: if you started a fresh session and read only the repo, would you know everything the previous session learned? If not, something is missing.

### Knowledge output

akari is a research institute, not a task runner. Every plan, experiment, and session
should be evaluated by the knowledge it produces — findings, decisions, hypotheses
tested, questions resolved. Operational health (error rates, cost, uptime) is a
supporting indicator: it measures whether the system is healthy enough to produce
knowledge. The fundamental efficiency metric is findings per dollar.

Before any implementation plan, ask: "What knowledge does this produce?" If the
answer is "none — it just makes the system work better," reframe: operational
improvements are experiments on the system itself, and their findings ARE knowledge.

**Inline logging checklist** (see `decisions/0004-inline-logging.md`):

1. Discovery of a non-obvious fact → write to project file **in the same turn**, before proceeding.
2. Config/env change → log entry with before/after and rationale, immediately.
3. Successful verification → log the exact command and output (not just "tested successfully").
4. Log incrementally throughout the session. A single end-of-session summary is a fallback, not the primary mechanism.
5. **Findings provenance:** Every numerical claim in an EXPERIMENT.md Findings section must include either (a) the script + data file that produces it, or (b) inline arithmetic from referenced data (e.g., "96/242 = 39.7%"). Claims without provenance are unverifiable and should be treated as suspect by downstream sessions.

## Autonomous execution

This repo runs autonomous agent sessions via [`infra/scheduler/`](infra/scheduler/README.md).
Each session follows the SOP at [docs/sops/autonomous-work-cycle.md](docs/sops/autonomous-work-cycle.md).
See [decisions/0005-autonomous-execution.md](decisions/0005-autonomous-execution.md) for rationale.

For governance (project priority, task selection, goal-directed planning, approval gates), see [docs/conventions/governance.md](docs/conventions/governance.md).

For session discipline (fire-and-forget, sleep limits, incremental commits, analysis throttling), see [docs/conventions/session-discipline.md](docs/conventions/session-discipline.md).

For enforcement layers (L0 code-enforced and L2 convention-only tables), see [docs/conventions/enforcement-layers.md](docs/conventions/enforcement-layers.md).

For task lifecycle tags, fleet-first task creation, and task decomposition, see [docs/conventions/task-lifecycle.md](docs/conventions/task-lifecycle.md).

Tasks are selected from project `TASKS.md` files. Priority order:
1. Tasks in the project recommended by /orient (respecting project priority)
2. Unblocked tasks (no `[blocked-by: ...]` tag)
3. When project budget is >90% consumed or exhausted, prefer `[zero-resource]` tasks
4. Tasks with concrete "Done when" conditions
5. Routine tasks before resource-intensive tasks
6. Tasks with explicit `Priority: high` before `Priority: medium` before untagged

### Approval gates

Autonomous sessions MUST NOT proceed with:
- **Resource decisions**: Requests to increase `budget.yaml` limits or extend deadlines
- **Governance changes**: Changes to approval workflow, budget rules, or other governance mechanisms in CLAUDE.md. Convention clarifications, gotcha additions, and skill improvements may be applied directly — they are verifiable and do not change governance. When in doubt, the test is: "Does this change what requires approval or how resources are allocated?" If yes, it's governance; write to APPROVAL_QUEUE.md.
- **Tool access**: Requests for tools, APIs, or model access not currently configured (see [decisions/0024](decisions/0024-tool-access-approval.md))
- **Production PRs**: Opening a pull request to any production module (`your-module-a`, `your-module-b`, `your-module-c`). Write to [`APPROVAL_QUEUE.md`](APPROVAL_QUEUE.md) with type `production-pr-request` including the module name, branch, and rationale. Do not execute `gh pr create` until approval is recorded. See [docs/design.md](docs/design.md) for the full procedure.

Git push does **not** require approval — sessions commit and push freely.

**Non-blocking approval items** — the following require an approval queue entry but do **not** block work:
- **GitHub releases and version tags**: Creating GitHub releases or version tags. Write to [`APPROVAL_QUEUE.md`](APPROVAL_QUEUE.md) and continue working. The heartbeat will notify a human; do not wait for approval before proceeding.

**Task-blocking approval items** — the following block the current task but not the session:
- **Tool access requests**: When a task requires a tool, API, or model not currently available. Write to [`APPROVAL_QUEUE.md`](APPROVAL_QUEUE.md) with type `tool-access`, tag the task `[blocked-by: tool-access approval for <tool>]`, and attempt to select a different task. If no other task is actionable, end the session.

For session-blocking items (resource decisions, governance changes, production PRs), write to [`APPROVAL_QUEUE.md`](APPROVAL_QUEUE.md) and end the session. The heartbeat will notify a human. Resume after approval is recorded.

Everything else does **not** need approval, including: infrastructure fixes (validation, tests, error handling, dry-run modes), new files, new decision records, schema changes, refactors — as long as correctness is verifiable by code (tests, type checks, validators, linters). Bug fixes and safeguards that prevent resource waste should be implemented immediately, not queued. The principle: if a future session can mechanically confirm the change is correct, human review is redundant as a gate (though it may still happen asynchronously).

### Session discipline

- Every autonomous session begins with /orient
- Every session ends with a git commit and log entry
- If a session produces no useful work (no actionable tasks, blocked on approvals),
  log that fact and end cleanly — do not invent work
- Do not re-litigate decisions recorded in `decisions/`. If a decision seems wrong,
  add it to APPROVAL_QUEUE.md as a structural request.
- **Sessions submit experiments, they do not supervise.** Never run training loops,
  rendering pipelines, or other long-running compute in-process within the agent session.
  This includes local GPU training (PyTorch, JAX, etc.) — even if it seems "quick,"
  training loops with multiple epochs routinely consume 5-20+ minutes and cause session
  timeouts. If a process will run longer than ~2 minutes, launch it via the experiment
  runner and register it with the scheduler for tracking:
  1. `python infra/experiment-runner/run.py --detach --project-dir <project-dir> --max-retries <N> --watch-csv <output-csv> --total <N> <experiment-dir> -- <command...>`
  2. `curl -s -X POST http://localhost:8420/api/experiments/register -H 'Content-Type: application/json' -d '{"dir":"<abs-path>","project":"<project>","id":"<experiment-id>"}'`
  **Mandatory flags for resource-consuming experiments** (per [decisions/0027-experiment-resource-safeguards.md](decisions/0027-experiment-resource-safeguards.md)):
  - `--project-dir`: enables budget pre-check AND post-completion consumption audit. Omitting this silently disables both safeguards.
  - `--max-retries`: must be explicit (never rely on default). Forces the launcher to decide how many retries are appropriate.
  - `--watch-csv` + `--total`: enables the retry progress guard. Without these, the runner cannot detect stalled or duplicate-producing retries.
  The registration step enables 10-second polling for completion notifications.
  Without it, the scheduler relies on slower discovery scans. After submitting,
  commit the experiment setup, log the submission, and end the session. Analysis
  of results happens in a future session.
  See [decisions/0017-no-experiment-babysitting.md](decisions/0017-no-experiment-babysitting.md).
- **Never sleep more than 30 seconds in a session.** Sleep-poll loops waiting for
  experiment output are waste. If you find yourself wanting to `sleep` and check
  output, you should be using fire-and-forget submission instead.
- **Commit incrementally during sessions.** After completing a logical unit of work
  (experiment setup, analysis write-up, log archiving, EXPERIMENT.md updates), run
  `git add && git commit` before proceeding to the next step. Do not defer all commits
  to the final "Commit and close" step. This prevents losing work if the session times
  out or exhausts its context/turn budget.
- **Batch archival commits per project.** When archiving (log entry archival, completed
  task archival), stage all archival changes for a single project together and commit
  once. Do not create separate commits for log moves vs. task moves within the same
  project.
- **Incremental analysis throttling.** When analyzing results from a running experiment,
  apply checkpoint discipline per [decisions/0023-incremental-analysis-throttling.md](decisions/0023-incremental-analysis-throttling.md):
  - Analyze at most at these checkpoints: ~25%, ~50%, ~75%, and 100% (final). Do not
    analyze on every session.
  - After an intermediate analysis, note in the task description when the next analysis
    is warranted (e.g., "Next analysis at ~N rows or completion").
  - If fewer than 20% new rows have accumulated since the last analysis, skip the task
    and select something else.
  - When creating an analysis task for a running experiment, split into a preliminary
    analysis task (satisfiable mid-experiment) and a final analysis task (blocked-by
    experiment completion).

### Enforcement layers

Conventions are enforced at different layers. The canonical L0/L2 tables live in [docs/conventions/enforcement-layers.md](docs/conventions/enforcement-layers.md). Read that file for the full list of what is code-enforced vs. convention-only.

Summary: **L0 (code-enforced)** items are automatically blocked or detected by scheduler code (`verify.ts`, `budget-gate.ts`, `fleet-scheduler.ts`, pre-commit hooks, etc.). **L2 (convention-only)** items require agents to self-enforce (session discipline, budget recording, temporal reasoning, etc.).

When adding a new enforcement mechanism, update [docs/conventions/enforcement-layers.md](docs/conventions/enforcement-layers.md) in the same turn.

### Task lifecycle tags

Use these tags in `TASKS.md` to coordinate across sessions:

- `[in-progress: YYYY-MM-DD]` — being worked on (prevents duplicate pickup)
- `[blocked-by: <description>]` — cannot proceed until condition is met. Use only for conditions requiring action outside the agent's control (human approval, external credentials, another task's completion, external team work). Implementation steps the agent can perform (installing packages, writing code, configuring tools) are NOT blockers — they are part of executing the task. **Before tagging:** (1) attempt the operation and cite the exact error message, (2) verify the error is genuinely external (not fixable within agent control). See relevant postmortem on unverified external blocker assumptions.
- `[blocked-by: external: <description>]` — waiting on external team work with uncertain timeline. When using this tag: (1) decompose the task to identify preparatory work that can proceed, (2) document pending external work in project README, (3) check for stale requests (7+ days) during orient. See ADR 0040.
- `[approval-needed]` — requires human sign-off before execution
- `[approved: YYYY-MM-DD]` — human approved, ready to execute
- `[zero-resource]` — task consumes no budget resources (analysis, documentation, planning)
- `[fleet-eligible]` — task can be assigned to fleet workers (GLM-5 on opencode backend). **This is the default for well-scoped tasks.** Apply unless the task fails the fleet-eligibility checklist below. Per ADR 0042-v2, ADR 0045.
- `[requires-opus]` — task requires Claude Opus capability (complex reasoning, multi-file refactors, ambiguous decisions). Not fleet-eligible. Fleet workers skip these tasks.
- `[escalate]` — fleet worker should escalate to Opus supervisor when encountering unexpected complexity, blockers not in task description, or quality concerns. Used during fleet execution, not during task definition.
- `[skill: <type>]` — skill-typed routing tag that classifies tasks by dominant capability (ADR 0062). Replaces binary fleet-eligible/requires-opus with finer-grained routing:

  | Skill type | Worker role | Model tier | Cost |
  |------------|-------------|------------|------|
  | record, persist, govern | Knowledge worker | GLM-5 | $0 (self-hosted) |
  | execute | Implementation worker | Best available* | Varies |
  | diagnose, analyze, orient, multi | Reasoning worker | Opus | API cost |

  **Skill definitions:**
  - `[skill: record]` → Knowledge management: documentation, status updates, archival
  - `[skill: persist]` → State management: cross-references, inventories, monitoring
  - `[skill: govern]` → Convention compliance: self-audits, tag validation, formatting
  - `[skill: execute]` → Implementation: code changes, scripts, bug fixes, tests
  - `[skill: diagnose]` → Root cause analysis: debugging, failure investigation
  - `[skill: analyze]` → Interpretation: experiment results, cross-project synthesis
  - `[skill: orient]` → Strategic: task selection, priority assessment, planning
  - `[skill: multi]` → Multi-factor: requires reasoning + implementation + knowledge

  **Backward compatibility:** `[fleet-eligible]` → `[skill: record]`, `[requires-opus]` → `[skill: multi]`. Both old tags remain valid. Skill tags are additive — prefer them for new tasks.

  *Implementation worker: currently falls through to Opus supervisor. When GPT-5.2, Composer, or Gemini arrive on opencode, EXECUTE tasks route to those models.

### Fleet-first task creation (ADR 0045)

**Every new task should be assessed for fleet-eligibility at creation time.** The fleet is akari's primary execution engine. Untagged tasks default to fleet-eligible and are assigned by the fleet scheduler. Only tasks explicitly tagged `[requires-opus]` are excluded from fleet assignment and reserved for the Opus supervisor. Apply skill tags (`[skill: record]`, `[skill: execute]`, etc.) for better routing precision.

**Prefer skill tags over binary classification.** Use `[skill: <type>]` tags (see above) for finer-grained routing. They're additive — a task can have both `[fleet-eligible]` and `[skill: record]`, but skill tags provide better routing precision.

**Fleet-eligibility checklist** — a task is fleet-eligible if ALL of these are true:
1. **Self-contained**: The task can be understood from its text + project README alone (no multi-file context gathering)
2. **Clear done-when**: The completion condition is mechanically verifiable
3. **Single concern**: The task does one thing (not "do X, then Y, then Z")
4. **No deep reasoning**: Does not require research synthesis, strategic decisions, or multi-step planning
5. **No convention evolution**: Does not modify CLAUDE.md, decisions/, or infra/ code

If ANY check fails → tag `[requires-opus]`. If ALL pass → tag `[fleet-eligible]`.

**Recommended tagging flow:**
1. Apply skill tag first: `[skill: record]`, `[skill: execute]`, etc. (see skill definitions above)
2. Add `[fleet-eligible]` if the task passes the checklist
3. Add `[requires-opus]` if it fails the checklist
4. Old tags alone still work, but skill tags enable better routing

**When in doubt, prefer `[fleet-eligible]`.** Fleet workers that encounter unexpected complexity can escalate via `[escalate]`. An optimistically-tagged task that gets escalated wastes one fleet session (~5 min, $0). An untagged task that sits in the queue wastes days of potential fleet throughput.

### Task decomposition

**Fleet-oriented decomposition** — when creating or reviewing tasks, actively decompose complex tasks into fleet-eligible subtasks. The fleet's value is proportional to the number of well-scoped tasks available. A single complex task that takes Opus 30 minutes is less efficient than 5 fleet-eligible subtasks that 5 GLM-5 workers complete in 5 minutes each.

**Decomposition triggers** — decompose a task when:
- It has more than 2 independent steps
- It touches more than 3 files
- It combines blocked and unblocked work
- It mixes mechanical work with judgment-requiring work

**Decomposition pattern:**
```
Before (single complex task):
- [ ] Run experiment on test set and analyze results
  Done when: Experiment results and analysis documented

After (fleet-decomposed):
- [ ] Set up experiment directory [fleet-eligible]
  Done when: EXPERIMENT.md with status: planned exists
- [ ] Write experiment script [fleet-eligible]
  Done when: Script runs without error on 1 sample
- [ ] Run experiment via experiment runner [fleet-eligible]
  Done when: Experiment submitted with progress.json
- [ ] Analyze experiment results [requires-opus]
  Done when: Findings section in EXPERIMENT.md with per-dimension scores
```

When a task covers multiple independent work streams with different dependency states, split it into independently actionable subtasks. If you split an upstream task (e.g., splitting "collect images" into "collect dataset-A" and "collect dataset-B"), check whether downstream tasks (e.g., "run generation on test images") should also be split. See relevant project postmortem on blocked tags stalling high-priority work for the incident that motivated this convention.

### Partial completion

Never mark a task `[x]` with a "(partial)" annotation. If work is partially done, keep the task `[ ]` and update the description to reflect remaining work, or split into a completed subtask and a new open task. The `[x]` checkbox is binary — the task selector treats it as complete and will never revisit it.

## Conventions

Convention details are extracted into individual files. Read the relevant file when working in that area.

| Convention | File | When to read |
|------------|------|-------------|
| File conventions | [docs/conventions/file-conventions.md](docs/conventions/file-conventions.md) | Creating or organizing files |
| Provenance | [docs/conventions/provenance.md](docs/conventions/provenance.md) | Citing papers, reporting results, creating literature notes |
| Temporal reasoning | [docs/conventions/temporal-reasoning.md](docs/conventions/temporal-reasoning.md) | Making claims about duration, age, or timelines |
| Production code discovery | [docs/conventions/production-code-discovery.md](docs/conventions/production-code-discovery.md) | Working with code in `modules/` |
| Decisions | [docs/conventions/decisions.md](docs/conventions/decisions.md) | Making significant choices, writing ADRs |
| Testing | [docs/conventions/testing.md](docs/conventions/testing.md) | Writing or modifying `infra/` code |
| Structured work records | [docs/conventions/structured-work-records.md](docs/conventions/structured-work-records.md) | Creating experiment directories, EXPERIMENT.md files |
| Resource constraints | [docs/conventions/resource-constraints.md](docs/conventions/resource-constraints.md) | Planning work that may consume budget resources |
| Model selection & provenance | [docs/conventions/model-selection.md](docs/conventions/model-selection.md) | Calling external models (LLM/VLM), documenting model choices |
| Creative Intelligence | [docs/conventions/creative-intelligence.md](docs/conventions/creative-intelligence.md) | Analyzing problems, reporting findings, designing experiments |

## Schemas

All schemas are minimal by design — add fields only when a real need arises. Schema templates are extracted into individual files. Read the relevant file when creating artifacts of that type.

| Schema | File |
|--------|------|
| Log entry | [docs/schemas/log-entry.md](docs/schemas/log-entry.md) |
| Task | [docs/schemas/task.md](docs/schemas/task.md) |
| SOP | [docs/schemas/sop.md](docs/schemas/sop.md) |
| Decision record | [docs/schemas/decision-record.md](docs/schemas/decision-record.md) |
| Experiment (design, directory, EXPERIMENT.md) | [docs/schemas/experiment.md](docs/schemas/experiment.md) |
| Budget & ledger (budget.yaml, ledger.yaml) | [docs/schemas/budget-ledger.md](docs/schemas/budget-ledger.md) |
| Literature note | [docs/schemas/literature-note.md](docs/schemas/literature-note.md) |
| Project README | [docs/schemas/project-readme.md](docs/schemas/project-readme.md) |

## Infrastructure

`infra/` holds shared tooling built by the group — experiment harnesses, data pipelines, experiment launchers, plotting utilities. Each tool gets its own subdirectory with a README following the same schema as projects.

Unlike `projects/`, infra directories contain source code. This is the one place in the repo where code lives. Projects consume infra as local imports; external users install from the respective infra subdirectory.

---
> Source: [victoriacity/openakari](https://github.com/victoriacity/openakari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
