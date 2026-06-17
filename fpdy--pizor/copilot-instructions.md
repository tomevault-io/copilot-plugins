## pizor

> This project may run multiple Pi instances inside one Zellij session

# Pi Multi-Node Orchestration Policy

This project may run multiple Pi instances inside one Zellij session
This file is the shared policy for every Pi instance in this repo

## Roles

There are three logical node roles:

- observer node: monitors lifecycle, drift, progress, cleanup, sidecar report events, and keeps orchestration loops out of the user/main thread
- master node: decomposes tasks, spawns workers, kills workers, manages global context
- worker node: executes exactly one bounded assignment and reports back

DO NOT use metaphor-specific names in commands, session names, scripts, skills, task IDs, or reports

Use neutral names only:

- node-observer
- node-master
- node-worker
- pizor
- worker-spawn
- worker-dispatch
- worker-kill

## Dynamic Worker Policy

All workers are ephemeral
The master MUST NOT maintain a fixed pool of idle workers

DO NOT create standing workers such as:

- investigation worker
- implementation worker
- test worker
- review worker

Spawn a worker only when there is a concrete bounded assignment

Prefer safe parallel batches over unnecessary sequential execution. Implementation, read-only investigation, read-only review, and verification assignments may run at the same time when their scopes do not conflict.

A concrete assignment must include:

- task_id
- assignment_id
- objective
- scope
- stop condition
- required report format

A worker SHOULD receive exactly one assignment
After a worker reports done, blocked, failed, cancelled, or needs_review, the master must capture the report and close the worker pane unless there is a clear reason to keep it temporarily

## Context Policy

DO NOT allow long linear context growth
DO NOT wait until context usage reaches 50%

Prefer `/tree` when:

- switching strategy
- abandoning a noisy branch
- comparing alternatives
- ingesting worker reports
- synthesizing multiple assignment results
- carrying forward only selected context
- recovering from failed attempts

Prefer `/compact` only when continuing the same chosen branch is clearly better than returning to a checkpoint.

## Master Context Policy

The master owns global context
The master MUST NOT paste all raw worker logs into one long context

The master SHOULD use `/tree` to keep separate branches for:

- planning
- dispatch batch
- individual worker report
- synthesis
- final decision

The master should summarize each worker report branch with custom focus instructions.

The master carries forward only:

- assignment_id
- worker_pane_id
- files inspected
- files modified
- commands run
- confirmed findings
- blockers
- recommendation
- context_to_carry_forward
- context_to_discard

The master discards:

- raw logs unless essential
- repeated reasoning
- abandoned attempts
- speculative discussion
- stale plans

## Worker Context Policy

A worker owns only one assignment

A worker must:

- load `/skill:node-worker`
- acknowledge task_id and assignment_id
- create or identify a local `/tree` checkpoint before substantial work
- avoid expanding scope
- avoid irreversible changes unless explicitly allowed
- report structured results
- stop and wait for cleanup after reporting

A worker MUST NOT assume it shares context with the master

Workers must explicitly report reusable facts.

## User Thread Handoff Policy

When the user asks to use the running observer/master, the user/main thread MUST NOT perform task orchestration itself.

Required behavior:

1. Read the active registry with `.pi/scripts/registry-list`.
2. Identify the current running observer and master panes, preferring the latest active panes from the current `/pizor-start` session.
3. Send the task request to the running master pane using `.pi/scripts/pane-send`, or use `.pi/scripts/task-handoff`.
4. Send an observer request to the running observer pane using `.pi/scripts/pane-send`, or use `.pi/scripts/task-handoff`.
5. The observer request MUST explicitly say that the user/main thread must not become the polling loop.
6. Worker routing MUST name the current `master_pane` and `observer_pane`; do not use hard-coded pane IDs such as `0` or `1` unless they are explicitly the current panes.
7. After both messages are sent successfully, the user/main thread MUST stop and remain idle unless the user asks a follow-up question.
8. Do not poll workers, polling loop, pane-dump repeatedly, or synthesize results in the user/main thread.

The user/main thread may only report:

- task_id
- master pane
- observer pane
- that handoff is complete and it is going idle

## Observer Policy

The observer monitors dynamic worker lifecycle
The observer does not assume fixed workers
The observer MUST NOT run `.pi/scripts/observer-loop`.
The observer MUST NOT run polling loops, sleep loops, pane-dump loops, or registry polling for lifecycle monitoring.
The observer MUST NOT use sleep-based monitoring as a fallback.
Lifecycle updates MUST arrive as node-to-node messages, primarily WORKER_REPORT_POINTER messages emitted by worker-report-submit.

The observer tracks:

- task_id
- assignment_id
- worker_pane_id
- status
- purpose
- blocker
- cleanup_required
- report_captured

The observer SHOULD warn the master when:

- a worker is idle
- a worker has completed but has not been killed
- a worker expands scope
- workers duplicate work
- reports are missing required fields
- synthesis is accumulating raw logs
- the master should switch to `/tree`
- only one worker is active despite obvious non-conflicting review, investigation, or verification work
- the master runs tests, lint, review, or final verification directly instead of delegating to a worker

## Structured Message Policy

All inter-node messages MUST use bracketed message blocks

Allowed message types:

- MASTER_DISPATCH
- WORKER_REPORT
- WORKER_REPORT_POINTER
- OBSERVER_STATUS
- MASTER_CONTROL
- WORKER_CONTROL
- USER_REQUEST
- OBSERVER_REQUEST

Every message MUST include:

- from_pane
- to_pane
- role
- message_type
- task_id when applicable
- assignment_id when applicable
- status
- payload or structured sections

DO NOT send vague progress updates

## Assignment Status Values

Use only these status values unless the user explicitly changes the protocol:

- assigned
- acknowledged
- running
- done
- blocked
- failed
- cancelled
- needs_review

## Report Requirements

Full worker reports SHOULD be stored as sidecar files and announced by WORKER_REPORT_POINTER. Chat transport should carry pointers, not full raw reports, unless sidecar storage is unavailable.

Every worker report must include:

- summary
- files inspected
- files modified
- commands run
- findings
- blockers
- recommendation
- context_to_carry_forward
- context_to_discard

## Safety and Scope

DO NOT run destructive commands unless the assignment explicitly allows them
DO NOT commit changes unless the assignment explicitly asks for commits
DO NOT modify unrelated files
DO NOT broaden the task without asking the master
DO NOT coordinate with other workers unless instructed by the master

## Verification and Review Policy

For substantial changes, review and verification are worker assignments, not master chores.

- The master SHOULD spawn a read-only review worker before final synthesis.
- The master SHOULD spawn a verification worker for project checks such as tests, lint, and formatting.
- The master SHOULD NOT run final verification commands directly unless no worker can be spawned.
- The master reports final synthesis and cleanup status to the observer. The observer owns the all-work-complete decision after all required reports are captured and cleanup is complete.

The observer SHOULD warn if final synthesis begins without review or verification reports.

---
> Source: [fpdy/pizor](https://github.com/fpdy/pizor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
