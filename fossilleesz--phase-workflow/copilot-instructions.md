## phase-workflow

> Use this skill for early-stage software projects that need phase-based planning, fixtures-first and tests-first implementation, scope control, change request tracking, verification, and handoff documents for continuing work across Codex sessions.


# phase-workflow

Use this skill to keep early-stage project work phase-based, testable, and recoverable across
Codex sessions.

## Mandatory Invocation

If `phase-workflow` applies, first open this `SKILL.md` in the current turn.
Then read only the needed reference file for the current decision.

Hook-injected reminders do not count as skill invocation. Project `AGENTS.md` rules do not
replace opening this skill when the skill applies. Do not rely on compressed chat history or
remembered `phase-workflow` rules.

Before mutating files, state the active authorization branch: plan-change, phase start gate,
post-gate execution, approach confirmation, status/handoff update, or recovery-record
mutation.

## Resource Navigation

Load only the resource needed for the current decision:

- Read `references/phase_policy.md` when starting, exiting, renumbering, or splitting a phase.
- Read `references/change_request_policy.md` when a new request appears during an active phase.
- Read `references/handoff_protocol.md` when resuming work in a new window or ending a work round.
- Read `references/verification_policy.md` before marking a phase complete or reporting test status.
- Copy from `templates/` when creating phase notes, handoff notes, TODOs, or decision records
  in a target project.
- Copy `templates/project_context_template.md` as `PROJECT_CONTEXT.md` when adopting an
  existing structured project.
- Use `examples/` only when the user needs a concrete pattern for a greenfield start, existing
  structured project adoption, change request, or new-window handoff.

## When To Use This Skill

Use this skill when the user is working on:

- A new project.
- An existing project with a clear structure that needs lightweight phase planning and handoff
  files.
- An MVP project.
- A Phase 0 initialization.
- A phase-based development plan.
- Scope control for a small or changing project.
- New-window handoff after prior Codex work.
- Mid-phase change request handling.
- Updates to `TODO.md`, `DECISIONS.md`, phase notes, or handoff documents.

## When Not To Use This Skill

Do not use this skill when:

- The user is only asking a simple question.
- The user explicitly asks for a direct small bug fix in a project that has not adopted
  `phase-workflow`, or explicitly opts out of `phase-workflow` for that request.
- The project is mature and already has a process the user does not want to change.
- The project state is unclear, heavily tangled, or lacks an identifiable entrypoint,
  verification command, or current goal.
- The current task does not need long-term context management.
- The repository has higher-priority development instructions.

In a project that has adopted `phase-workflow`, a direct small bug fix is still governed by
phase-workflow. Classify it as Phase X.2 unless the user explicitly opts out of phase-workflow
for that request. An opt-out does not rewrite project workflow records; it only scopes that
request outside the workflow.

## Core Workflow

For each phase:

1. Lock phase scope.
2. Prepare fixtures/examples.
3. Write failing tests.
4. Implement the minimal capability.
5. Verify with actual commands.
6. Update `TODO.md`, phase notes, and handoff/current-state.
7. Move to the next phase only after verification.

## Authorization Model

Use this single authorization model before applying the detailed rules below. When two signals
conflict, use the narrower authorization and stop at the next required gate.

| Signal | Authorizes | Does not authorize | Required stop |
| --- | --- | --- | --- |
| Plan-change confirmation | planning-file updates only | Tests, code, scripts, migrations, restores, or other technical files | Stop after planning files |
| Phase start request | phase analysis and visible start gate only | Execution, file edits, or current-phase mutations | Stop after the gate |
| Post-gate execution confirmation | current-phase technical mutations that pass mutation preflight | Next-phase work, multi-phase execution, planning-file mutations, recovery-record mutations, or unstated approach choices | Stop when the current phase exits |
| Approach confirmation | approved non-trivial approach inside the current phase | Phase boundary changes or broader scope | Stop if the approach is rejected |
| Mutation preflight pass | The checked current-phase mutation branch | Failed-gate, unconfirmed, wrong-branch, next-phase, or multi-phase mutations | Stop before files if any check fails |
| Phase violation detected | read-only recovery audit and chat-visible incident report before user choice; recovery record repairs after user choice | New implementation, next-feature work, continuing the violated flow, or pre-choice record repair | Stop before record repairs until the user chooses keep-and-audit or rollback |

This model keeps plan-change confirmation, phase start requests, post-gate execution
confirmation, approach confirmation, mutation preflight, and phase violation recovery as
separate controls. They cannot substitute for each other and do not authorize multi-phase
execution. A successful authorization does not authorize next-phase work and does not
authorize multi-phase execution.
Do not use the phase violation row to write recovery record repairs before user choice.

## Workflow Authority

When `phase-workflow` applies, it is the highest phase-boundary and authorization workflow for
the work. It controls scope, authorization, phase boundaries, and file-modification
boundaries.

Other workflow behaviors can support execution, but they do not control phase boundaries.
This includes plan executors, test workflows, debugging workflows, refactoring workflows,
migration workflows, code generation helpers, documentation generators, task checklists, and
tool-specific workflows. These workflows can only guide work inside the currently authorized
phase.
They cannot decide phase boundaries, advance to the next phase, or execute multiple phases in
one run.

If a user provides a plan with multiple phases, multiple verification loops, or multiple
independent outputs, even a request such as "implement this plan" only authorizes the
currently confirmed phase. Only the currently confirmed phase may execute. Unconfirmed phases
must stop at a phase gate.

Todo lists, checklists, progress trackers, and `update_plan` cannot substitute for phase start
gates, split confirmation, approach confirmation, or post-gate execution confirmation.

Do not record batch phase completions such as `Phase 1-5 complete`, `Phase 2/3/4`, or
`multi-phase migration complete`; each phase must have separate status, verification, and
handoff records.

## Mutation Preflight

Before repository mutations, run a mutation preflight. Repository mutations include file
writes, deletes, moves, restores, generated tests, generated docs, scripts, migrations, and
implementation changes. Any work that changes repository files must pass mutation preflight
before modifying files.

Mutation Authorization Branches:

- A planning-file mutation requires plan-change confirmation and planning-file scope only; it
  does not require post-gate execution confirmation.
- A recovery audit before user choice is read-only and may produce only a chat-visible
  incident report.
- A recovery-record mutation requires Phase Violation Recovery and the user's keep-or-rollback
  choice. After the user chooses, recovery-record mutations may repair `PLAN.md`, `TODO.md`,
  phase notes, and handoff records.
- A status/handoff-record mutation requires current phase execution or phase exit context. It
  can update `TODO.md`, phase notes, handoff notes, and `DECISIONS.md` when recording a
  durable decision already made inside the authorized phase or phase exit context.
  It must record actual verification results when claiming completion or test status. If
  verification has not run, record that status honestly and do not claim completion or passing
  tests. It can update blocker, unfinished status, handoff, or current-state records without a
  fresh verification command when no completion or test status is claimed. It does not
  authorize plan changes, new strategy decisions, technical implementation, or recovery
  repair.
- A technical mutation requires visible phase start gate, post-gate execution confirmation,
  and mutation preflight.
- If the mutation type is unclear, stop before mutating files.

Post-gate execution confirmation authorizes technical mutations only. It does not authorize
planning-file mutations or recovery-record mutations. Planning-file mutations use plan-change
confirmation. Recovery-record mutations use Phase Violation Recovery.

Classify mutation branches by change intent and content, not by file name alone. `TODO.md`,
phase notes, and handoff notes can contain planning-file mutations or status/handoff-record
mutations. Scope, phase boundary, active task, acceptance criteria, or roadmap changes are
planning-file mutations. Verification results, current status, and handoff summaries are
status/handoff-record mutations.

Do not apply technical mutation gate checks to planning-file mutations or recovery-record
mutations; visible phase start gate and post-gate execution confirmation are technical
mutation checks.

Planning-file mutation checks:

- What planning change has the user confirmed?
- Is every changed file a planning file or planning note?
- Will the work stop after planning-file updates?
- Would this mutation complete multiple phases at once?

Recovery-record mutation checks:

- Is Phase Violation Recovery open?
- Has the user chosen keep implementation and backfill audit, or roll back selected changes?
- Is the mutation limited to repairing planning, phase, and handoff records?
- Does the mutation avoid new implementation?

Status/handoff-record mutation checks:

- Is the update reporting current phase execution, verification, blocker, unfinished status,
  handoff, or current-state status?
- Is it limited to `TODO.md`, phase notes, handoff notes, or a narrow `DECISIONS.md` update?
- Is any `DECISIONS.md` update limited to recording a durable decision already made inside the
  authorized phase or phase exit context?
- Are actual verification results recorded when completion or test status is claimed?
- If verification has not run, does the record say so without claiming completion or passing
  tests?
- Does it avoid plan changes, technical implementation, and recovery repair?

Technical mutation checks:

- What is the current phase?
- Has the visible phase start gate been shown?
- Is there a separate post-gate execution confirmation?
- Does this mutation belong only to the current phase?
- Would this mutation complete multiple phases at once?

If any answer fails, stop before mutating files. Do not use one mutation to complete multiple
phases; split the work or return to the appropriate phase gate.

Phase 0.2 is a planning baseline only. Phase 0.2 allows planning records only: `PLAN.md`,
`TODO.md`, phase notes, handoff notes, planning baselines, roadmaps, and candidate Phase 1
goals. During Phase 0.2 there are no
tests, code skeletons, migrations, file restores, technical implementation, scripts, or
non-planning technical documentation. Non-planning technical documentation includes API docs,
architecture docs, migration guides, usage docs, module docs, or generated technical docs. Do
not treat planning records as technical deliverables. In short: no tests, code skeletons,
migrations, file restores, or technical implementation. After Phase 0.2, stop before Phase 1.

Plan-change confirmation only authorizes planning-file updates, then stop. It does not
authorize tests, code, scripts, migrations, restores, or other technical files.

## Phase Violation Recovery

Use Phase Violation Recovery when an approved phase boundary has already been crossed. This is
a mandatory recovery flow, not permission to continue implementation.

When a phase violation is discovered:

1. Stop new implementation immediately.
2. Do not continue to the next feature.
3. Do not keep implementing while recovery is open.
4. Before the user chooses keep-and-audit or rollback, recovery audit is read-only.
5. Audit files and records changed outside the approved phase boundary as a read-only step.
6. Report a chat-visible incident report that marks `implementation happened outside approved
   phase boundary`.
7. The chat-visible incident report should record which files, tests, migrations, restores,
   docs, and status records were affected.
8. Ask the user to choose: keep implementation and backfill audit, or roll back selected
   changes.
9. Do not write recovery record repairs before the user chooses.
10. After the user chooses, repair `PLAN.md`, `TODO.md`, phase notes, and handoff records.
11. Make the state honest and recoverable so a new Codex session can resume without chat
   history.

## Operating Rules

- Keep one phase focused on one closed loop.
- Before starting a major phase or sub-phase, output a visible phase start gate.
- The phase start gate must include the current phase, goal, non-goals, split decision,
  verification loop, and confirmation status.
- Before starting a major phase, report whether the scope is one single verification loop.
- A request to start a phase is not confirmation. It only authorizes phase analysis and the
  phase start gate.
- Do not treat "start Phase X" as confirmation. Do not treat "begin Phase X", "continue
  Phase X", "enter Phase X", or similar English requests as execution confirmation either.
- Do not write `Confirmation status: received in your current request` unless the current
  message is a clear confirmation after the gate has already been shown.
- Always stop after reporting the phase start gate. Execution requires a separate user response
  after the phase start gate is displayed.
- After plan-change update, a later request to start, continue, enter, or execute an
  already-recorded Phase X.N only authorizes the phase start gate; it does not authorize
  execution. The start gate must stop before execution.
- Plan-change confirmation, phase start request, and post-gate execution confirmation cannot
  substitute for each other. Post-gate execution confirmation authorizes technical mutations
  only; it does not authorize planning-file mutations or recovery-record mutations.
- Plan Mode is optional. When Plan Mode is enabled and this skill applies, Codex must activate
  and follow phase-workflow first.
- Plan Mode cannot bypass phase-workflow activation, phase boundary change checks, phase
  start gates, or post-gate execution confirmation.
- In Plan Mode, plan, phase, scope, or design-change requests must go through
  `phase-workflow` classification before returning a proposed plan.
- Plan Mode proposed plan cannot substitute for plan-change confirmation, phase start request,
  and post-gate execution confirmation. Plan Mode does not authorize execution.
- If the user gives an ambiguous response, ask a short explicit confirmation question and do
  not edit files.
- If the target folder is an empty folder, treat Phase 0 initialization as gated work.
- Before Phase 0, classify folder state as `empty folder`, `existing structured project
  candidate`, `existing workflow project`, or `unclear project state`.
- Folder state classification is only a recommendation. It must be reported with evidence and
  does not authorize execution.
- For an existing structured project candidate, use Phase 0.1 for workflow adoption and
  Phase 0.2 for a planning baseline.
- For an existing workflow project, recover compact current state from project files before
  proposing the next phase gate.
- During Phase 0.1, create or fill workflow files and `PROJECT_CONTEXT.md`; do not overwrite
  existing project files, plan a roadmap, modify application code, refactor, or start a cleanup
  campaign.
- `AGENTS.md` is a Phase 0 or Phase 0.1 adoption output. Do not overwrite an existing
  `AGENTS.md`; append a small `phase-workflow` section. If a `phase-workflow` section already
  exists, do not add it again. Users may explicitly request refreshing the `phase-workflow`
  section after updating this skill; treat that as a maintenance action, not a per-prompt
  action.
- `.codex/hooks.json` is a Phase 0 or Phase 0.1 adoption output only when optional
  project-level hook support is selected or explicitly requested. Do not overwrite an existing
  `.codex/hooks.json`; merge only the `phase-workflow` `UserPromptSubmit` entry. If a
  `phase-workflow` hook entry already exists, do not add it again. A conflicting hook command
  or hook location requires user confirmation before replacement. Users may explicitly request
  refreshing the `.codex/hooks.json` hook entry after updating this skill; treat that as a
  maintenance action, not a per-prompt action. When creating or refreshing the optional hook
  entry, use the target project's absolute path to `hooks/phase_workflow_prompt.py` in the
  hook command. Do not rely on the hook runner's current working directory. Restart Codex and
  re-review/trust the hook after changing the absolute hook command. The hook does not
  generate `AGENTS.md`, update `.codex/hooks.json`, scan project files, restore state, or
  invoke the skill directly.
- During Phase 0.2, use `PROJECT_CONTEXT.md` and a user-confirmed next project goal to create
  a rough planning baseline; do not infer the roadmap from code observations alone.
- Phase 0.1 completion does not authorize Phase 0.2. Phase 0.2 completion does not authorize
  Phase 1. Every phase and sub-phase requires its own visible start gate and separate user
  confirmation.
- Output a visible Phase 0 start gate before creating any project files.
- Do not create baseline workflow files before user confirmation. Baseline workflow files
  include README.md, AGENTS.md, PLAN.md, TODO.md, and DECISIONS.md.
- Split when the user requests a split, when there are multiple independent outputs,
  unrelated user-visible capabilities, separate verification loops, or when reviewability,
  transparency, phase size, risk, user confidence, or avoiding opaque large phases makes the
  phase easier to supervise.
- Do not split only because one coherent phase has several mechanical tasks, but do split when
  the user wants a large phase broken down for review.
- Do not automatically start the next phase; wait for explicit user confirmation.
- Do not create fixtures, tests, or implementation changes before user confirmation of the
  phase boundary.
- Prefer fixtures/examples before implementation.
- Prefer failing tests before changing behavior.
- Implement only the minimum capability needed for the active phase.
- Record scope changes before implementing them.
- Run actual verification commands before marking work complete.
- Update project files so the next Codex session can resume without chat history.
- Follow repository instructions when they are more specific than this skill.

## Plan-First Execution

Planning records are execution inputs, not after-the-fact summaries.

For scope-changing work, update planning files before technical files. This applies to new
phases or sub-phases, phase splits, New Major Phase work, Phase 0.1 workflow adoption,
Phase 0.2 planning baseline, route changes, changes to the current phase goal, non-goals,
outputs, acceptance criteria, or any request that changes the current `PLAN.md` or `TODO.md`
task boundary.

Planning files include `PLAN.md`, `TODO.md`, and, when relevant, the active phase note,
handoff note, `PROJECT_CONTEXT.md`, or `DECISIONS.md`.

Do not start fixtures, tests, implementation, or other technical file changes until the
relevant plan record exists. Do not use `PLAN.md` or `TODO.md` as after-the-fact summaries for
scope-changing work. Prohibited order: Implement first, document plan later.

Current-scope implementation details, verification results, actual modified file lists,
phase note updates, handoff summaries, and small non-scope-changing corrections can be
recorded after the technical work.

## Reviewable Splits And Approach Confirmation

Any phase split is a phase boundary change, regardless of whether the split is requested by
the user or recommended by Codex. Use one split flow for both sources. When a split is needed,
output a split interpretation or phase boundary change proposal first, ask the user to confirm
the planning change before updating planning files, then update `PLAN.md`, `TODO.md`, and the
active phase note. After those planning files are updated, stop. Split confirmation only
authorizes planning-file updates, then stop; it does not authorize Phase X.N or any
implementation work.

Phase boundary change confirmation applies to splits, any one-decimal Phase X.N sub-phase,
New Major Phase work, Backlog moves, and phase goal, non-goal, acceptance criteria, or
verification loop changes. Phase X.1 and Phase X.2 are common examples, not the complete
boundary. When Codex recommends a split, it must propose the complete currently visible
sub-phase structure before asking for confirmation. The proposal must assign Phase X.1, Phase
X.2, and later one-decimal numbers to known follow-up sub-phases, and mark uncertain or
out-of-scope work as later phase work or Backlog. Do not create only the immediate next Phase
X.1 and route the user directly to the Phase X.1 start gate. When Codex proposes a new Phase
X.N, it must explain why that sub-phase is needed, show a phase boundary change proposal, and
ask whether to update the plan. After confirmed planning-file updates, Codex stops before
showing the new phase or sub-phase start gate. Show the already-recorded Phase X.N start gate
only after a later user request. Use one decimal level only; do not introduce Phase X.N.M.

After plan-change update, that later request still only authorizes the phase start gate. It
does not authorize execution. The start gate must stop before execution and wait for
post-gate execution confirmation.

Plan Mode is optional, but when it is enabled for a task covered by this skill, Codex must
activate and follow phase-workflow first. Plan Mode cannot bypass phase-workflow activation,
phase boundary change checks, phase start gates, or post-gate execution confirmation. Run the
same classification before returning a proposed plan.

Plan Mode proposed plan cannot substitute for plan-change confirmation, phase start request,
and post-gate execution confirmation. Plan Mode does not authorize execution.

Valid split reasons include reviewability, transparency, phase size, risk, user confidence,
avoiding opaque large phases, multiple independent outputs, unrelated user-visible
capabilities, or separate verification loops.

Approval model:

> Split confirmation controls phase boundaries. Phase confirmation controls execution permission. Approach confirmation controls how non-trivial work will be done.

Approach confirmation applies to any phase or sub-phase. If a phase or sub-phase has a
non-trivial execution approach choice, show the proposed execution approach and wait for user
confirmation before modifying related files. A phase confirmation does not approve an unstated
approach.

Non-trivial execution approach choices include algorithms, architecture, libraries or
dependencies, data structures, Markdown document structure, prompt rewrite strategy, template
field design, policy semantics, test strategy, migration or compatibility strategy, and
user-visible output format. Documents, prompts, templates, policies, tests, and code can all
require approach confirmation when the approach affects outputs, maintainability, validation,
or long-term behavior.

If the user rejects the proposed approach, stop execution. Update `TODO.md`, `PLAN.md`, or the
active phase note as needed, show a revised approach confirmation, and wait for user
confirmation before continuing.

Do not treat split confirmation as execution approval. Do not treat phase or sub-phase
confirmation as approval for an unstated approach. Do not continue implementation after the
user rejects the approach.

## Phase Start Checklist

Before creating fixtures, tests, implementation changes, or any project files for Phase 0,
output a phase start gate and wait for user confirmation.

Identify:

- Current phase.
- Whether the previous phase is complete.
- Folder state: `empty folder`, `existing structured project candidate`, `existing workflow
  project`, or `unclear project state`.
- Current phase goal.
- Current phase non-goals.
- Split decision: keep the current phase, split into Phase X.N, move to a New Major Phase, or
  place in Backlog.
- Whether the phase is still one single verification loop.
- Proposed execution approach, when the work has a non-trivial approach choice.
- Inputs.
- Outputs.
- Acceptance criteria.
- Work that is not allowed in this phase.
- For Phase 0, baseline workflow files to create.
- For Phase 0.1, detected existing files, files to create or fill, files not to overwrite,
  verification command, and `PROJECT_CONTEXT.md` fields.
- For Phase 0.2, `PROJECT_CONTEXT.md`, user-confirmed next project goal, files to update, and
  candidate Phase 1 goal.
- Whether user confirmation has been received.
- Confirmation source: a separate user response after the phase start gate is displayed.
- If the response is unclear or ambiguous, ask a short explicit confirmation question before
  editing files.

If the boundary is unclear, stop and clarify the phase before editing files.

## Phase Exit Checklist

Before marking a phase complete, confirm:

- Tests pass.
- Smoke test passes, when applicable.
- Outputs match the current phase agreement.
- `TODO.md` is updated with current-only tasks.
- Phase note records historical completion details.
- Compact handoff/current-state note is updated.
- `DECISIONS.md` records durable decisions only.
- No unexplained scope expansion occurred.
- Verification results and the recommended next action have been reported.
- User confirmation is received before starting the next phase.

## Change Request Policy

Classify mid-phase changes before implementing them:

- One-decimal current-phase sub-phase: Phase X.N.
- Common small scoped addition example: Phase X.1.
- Common current phase bug fix example: Phase X.2.
- New capability or separate verification loop: New Major Phase.
- Valuable but not now: Backlog.
- MVP expansion idea: record only, do not implement directly.

Every change request should include impact on files, tests, outputs, and acceptance criteria.
If the classification is New Major Phase, update the plan and wait for user confirmation before
implementing it.

## Handoff Policy

Do not rely on chat history. Recover context from project files.

At the start of a new window, read compact recovery context in this order:

1. Read `AGENTS.md` first.
2. Read only the first 80-120 lines of the latest compact handoff or current-state file.
3. Read only the current phase, active task, blocked, and next task sections of `TODO.md`.

Do not read full `PLAN.md`, `DECISIONS.md`, `PROJECT_CONTEXT.md`, full `TODO.md`, full
handoff files, or full phase notes by default. Treat `PLAN.md`, `DECISIONS.md`,
`PROJECT_CONTEXT.md`, full `TODO.md`, full handoff files, and phase notes as targeted or
on-demand recovery sources.

Use `rg` to inspect `PLAN.md`, `DECISIONS.md`, `PROJECT_CONTEXT.md`, or phase notes only when
compact state is missing, conflicting, or explicitly requested. Read older entries only when
scope changes, conflicts, missing verification, unclear decision sources, or explicit history
requests require them.

The compact handoff/current-state file is not a complete history, audit log, phase table, or
`PLAN.md` summary. Its first 80-120 lines should contain a project summary, current phase,
recently completed work, last verification, blockers, next action, and do-not-do items.
Historical completed task lists belong in phase notes, not in the default `TODO.md` recovery
area.

`PROJECT_CONTEXT.md` is an adoption/background baseline, not a routine status file. Update it
only when stable project identity, directory responsibilities, verification commands, or
durable boundaries change.

`DECISIONS.md` is for durable decisions only. Do not record ordinary phase completion,
verification logs, or execution history there.

`DEV_LOG.md` is not a baseline workflow file, default recovery source, end-of-round record, or
recovery repair target. Legacy `DEV_LOG.md` files may remain in adopted projects, but the
workflow no longer requires creating, reading, or updating them.

Phase-exit and end-of-round record updates use the same bounded context rule. Do not read full
`PLAN.md`, `TODO.md`, `DECISIONS.md`, `PROJECT_CONTEXT.md`, full handoff files, or full phase
notes just to update status records. For `TODO.md`, read and update only the current phase,
active task, blocked, next task, and do-not-do sections. For `PLAN.md`, use `rg` or
heading-scoped reads to locate the relevant phase only when phase boundaries, scope, or
roadmap entries change. For `PROJECT_CONTEXT.md`, use scoped reads only for adoption or stable
baseline changes. For handoff and phase notes, read only the relevant heading or the first
80-120 lines unless a conflict requires more context.

At the end of each development round, leave a recoverable state by updating the active TODOs,
phase note, and compact handoff/current-state note. Update DECISIONS only for durable
decisions.

## Verification Policy

Run actual commands before claiming completion.

Default test command:

```bash
python -m pytest -q
```

Rules:

- Do not use "should run" as a completion standard.
- Do not mark a phase complete when verification fails.
- Record the actual command and result in the phase note or handoff note.
- If a CLI exists, include a smoke test for the CLI.

---
> Source: [FossilLeeSZ/phase-workflow](https://github.com/FossilLeeSZ/phase-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
