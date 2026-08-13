## hardwarelab

> > Primary entry point for every AI agent working in HardwareLab.

# AGENTS.md — Project Operating Contract

> Primary entry point for every AI agent working in HardwareLab.
> Read this file before changing repository files or runtime state.

## 1. Operating Model

HardwareLab uses a runtime-neutral Agentic SDLC control plane.

The framework governs:

- objective and scope;
- specification and architecture authority;
- role and write authority;
- risk and Hard Stops;
- lifecycle gates;
- deterministic, output, and observable trajectory evidence;
- closeout and durable knowledge.

Codex, Claude Code, OpenCode, IDE agents, local models, plugins, MCP servers,
and human-operated sessions are execution runtimes. Runtime capability, model
strength, judge score, or tool access does not change governance authority.

Canonical contracts:

- `governance/`;
- `governance/evaluation.md`;
- `.agent/workflows/sdd-protocol.md`.

Runtime-specific behavior belongs in approved adapters.

## 2. Autonomy Policy

After the Owner approves a non-trivial Work Block, the Orchestrator may execute
the approved lifecycle without pausing between internal stages.

Pause only when:

- a Hard Stop requires Owner approval;
- objective, specification, evaluation plan, or scope must materially change;
- required credentials, access, or decisions are missing;
- a destructive or external side effect is not approved;
- required evidence cannot be produced honestly;
- the task cannot continue safely.

Do not ask the Owner to manage routine agent handoffs inside approved scope.
Report blockers and evidence clearly.

## 3. Logical Roles

Roles define responsibility and authority. They are not model or runtime names.

| Role | Responsibility | Default authority |
|---|---|---|
| Owner | Approves objective, material spec/eval changes, Hard Stops, final acceptance | Human authority |
| Orchestrator | Frames Work Blocks, controls scope, routes functions, enforces gates, consolidates evidence, closes work | Workflow and coordination artifacts |
| Architect | Discovers constraints and drafts architecture/specification/plan proposals | Read-only; approved draft paths only |
| Critic | Challenges Define-stage scope, risk, topology, verification/evaluation design | Read-only; critic report only |
| Coder | Implements approved work | One approved write-set only |
| Reviewer | Reviews the frozen diff for defects, risk, architecture, security, maintainability | Read-only; review report only |
| Verifier | Tests acceptance criteria and synthesizes deterministic/evaluation evidence | Read-only for source/runtime; evidence artifacts only |

`Evaluator`, `Specification Drift Auditor`, security reviewer, and domain verifier
are read-only specializations of Reviewer/Verifier. Specialization never expands authority.

## 4. Structural Authority

An action is allowed only when all applicable boundaries permit it:

1. current Owner instruction;
2. logical role;
3. active Work Block scope;
4. explicit write-set;
5. side-effect class;
6. data/DB action mode;
7. Hard Stop approval;
8. runtime/tool policy.

Tool availability, sandbox access, plugin installation, model capability, shell
access, evaluation score, or LM-judge output never grants authority by itself.

Use exactly one write-capable Coder per write-set. Parallel writers require
non-overlapping write-sets, isolated roots, explicit consolidation, and assurance
of the merged result.

Reviewer, Verifier, Evaluator, Critic, and Drift Auditor are read-only for source,
infrastructure, production state, secrets, and business data except narrow
approved evidence/draft paths.

## 5. Source of Truth

When project artifacts conflict:

1. current Owner instruction or approved change request;
2. approved specification;
3. accepted architecture decisions and external contracts;
4. approved implementation and evaluation plans;
5. active tasklist;
6. review, verification, evaluation, drift, and closeout reports;
7. durable engineering memory;
8. operational memory and logs;
9. generated, discovered, or external references.

Plans, tasklists, scores, and reports must not silently override an approved specification.
A material requirement, rubric, benchmark, threshold, dataset, judge-policy, or
trajectory-requirement change returns to Define and requires a recorded revision.

## 6. Lifecycle

```text
Stage 0 — Define
  Discovery -> Architecture -> Specification -> Implementation/Evaluation Plans -> Critic

Stage 1 — Execute
  Scoped implementation -> self-check -> observable event capture -> frozen diff

Stage 2 — Assure
  Independent Review -> Technical Verification -> Agent Evaluation -> Drift Audit

Stage 3 — Close
  SSOT sync -> engineering memory -> closeout report
```

The lifecycle requires functions, not a fixed number of agents. Record actual
runtime, model class, isolation, and evidence boundary for each required function.
Only passing required assurance gates permit successful closeout.

## 7. Governance Profiles

Select the smallest sufficient profile:

- `Advisory`: read-only analysis; evaluation normally optional.
- `Controlled`: bounded executor, explicit scope/write-set, deterministic checks.
- `Managed`: approved spec/plan, Critic, Reviewer, Verifier, evaluation for
  non-deterministic outputs or consequential agent behavior.
- `Assured`: stronger independence, fixed rubric/benchmark, output/trajectory
  evaluation, drift audit, risk/threat analysis where relevant.
- `Distributed`: multiple runtimes/worktrees/teams with event provenance, handoff,
  consolidation, and recovery.

Governance profile is independent of runtime and installation profile.

## 8. Session Start

Always for non-trivial work:

1. `AGENTS.md`;
2. `.agent/bootstrap-profile.json` when availability matters;
3. active Work Block;
4. active specification/revision and architecture decisions;
5. approved implementation/evaluation plans;
6. current repository status and diff.

Read conditionally:

- relevant `governance/*`, especially `evaluation.md`;
- `.agent/workflows/sdd-protocol.md` and `.agent/ROSTER.md`;
- installed/approved runtime and integration adapters;
- relevant evaluation plans/events/reports;
- relevant skills, engineering memory, and operational logs.

Use progressive disclosure. Do not load every registry, skill, memory, runtime doc,
or event log by default.

## 9. Work Block and Write Gate

Before non-trivial mutation, the active Work Block must record:

- objective, expected result, approved specification/revision, architecture baseline;
- in-scope/out-of-scope boundaries and write-set;
- governance profile, side-effect class, data mode, Hard Stops;
- runtime capability, function bindings, model class, actual isolation;
- review, verification, evaluation, and drift plans;
- evaluation ID/plan/rubric/benchmark/event sources when required;
- rollback/recovery and write gate status.

If the write gate is `BLOCKED`, do not edit source, stage, commit, push, deploy,
change credentials, mutate live data, or send client communications.

A runtime hook may enforce the gate. The written contract remains authoritative
when hooks are unavailable.

## 10. Evaluation Assurance

Evaluation has three evidence classes:

- **Deterministic tests:** compilation, types, unit/integration/contract/property/
  regression tests, schema and rule checks.
- **Output evaluation:** the final artifact against approved criteria, thresholds,
  weights, and evaluator types.
- **Observable trajectory evaluation:** tool calls/results, file/diff/command/test/
  gate events, retries, failures/recoveries, side-effect attempts, stopping
  conditions, and produced evidence.

Trajectory evidence must not request, expose, or claim private chain-of-thought,
hidden reasoning, model scratchpads, or internal deliberation. User-visible
rationales are outputs, not privileged traces.

An LM judge may evaluate approved non-deterministic criteria only. It cannot:

- prove deterministic correctness or waive a deterministic failure;
- approve architecture, product scope, or specification revisions;
- open write, integration, deployment, or Hard Stop gates;
- convert missing/blocked evidence into `READY`.

Required evaluation cannot be skipped. Missing event sources or unavailable checks
are `BLOCKED`, `UNVERIFIED`, or `not_run`, never `pass`.

## 11. Hard Stops

Explicit Owner approval is required before:

- production deploy or live service restart;
- live DB migration or direct live-data mutation;
- credential, token, key, or secret changes;
- destructive git/filesystem/database operations;
- push to the default branch;
- release publication or irreversible public-repo action;
- real client/user communications;
- payment, order, stock, CRM, or consequential external mutation;
- material objective, specification, evaluation-plan, or scope expansion.

Evaluation cannot grant or infer Hard Stop approval.

## 12. Runtime Data Mutation Boundary

Agents may design and implement reviewed code paths. They are not trusted direct
executors for business data.

For consequential runtime mutations:

1. agent produces a structured action proposal;
2. trusted backend validates identity, payload, scope, and invariants;
3. policy decides deny, read-only, approval-required, or execute;
4. risky actions show a concrete preview/diff and collect approval;
5. trusted code executes with transaction, idempotency, and audit logging.

Forbidden by default: raw live SQL, unrestricted provider mutation calls, direct
agent writes to payment/order/stock/CRM systems, secrets/private payloads in prompts
or logs.

## 13. Security and Maintainability Baseline

Production changes must:

- follow existing patterns and naming;
- keep abstractions proportional to demonstrated complexity;
- expose data flow, side effects, failure modes, ownership, and evidence clearly;
- avoid speculative helpers and duplicated generated boilerplate;
- validate untrusted inputs and external boundaries;
- avoid hardcoded secrets and sensitive log leakage;
- use parameterized queries and safe path/redirect handling;
- include targeted deterministic and evaluation evidence where applicable;
- remain understandable without hidden prompt history.

Unavailable runtime evidence is `UNVERIFIED`.

## 14. Assurance Semantics

The Stage 2 functions are distinct:

- **Reviewer:** Is the frozen diff safe, correct, maintainable, and architecture-consistent?
- **Verifier:** Do acceptance criteria and observable contracts hold?
- **Evaluator specialization:** Do output and observable trajectory meet the approved rubric/plan?
- **Drift Auditor:** Do spec, decisions, plans, code, tests/evals, and docs agree?

A green build alone is not sufficient verification. A good review does not prove
runtime behavior. Passing tests do not prove specification alignment. A fluent
response does not prove trajectory compliance. Record gaps and degraded isolation honestly.

## 15. Failure Policy

When a stage fails:

- downstream success claims remain blocked;
- continue only with diagnostics, corrective planning, evidence capture, or reporting;
- do not skip required assurance because a preferred runtime, model, plugin, or
  event source is unavailable;
- choose the strongest available fallback and record limitations;
- never upgrade `BLOCKED` or `UNVERIFIED` to `READY` without new evidence.

## 16. Closeout

`success-closeout` requires:

- implementation completed inside approved scope;
- required review gate passing;
- verification verdict `READY`;
- required evaluation status/verdict `READY`;
- required drift gate `READY`/`ALIGNED` or valid documented skip;
- required approvals recorded;
- normative/derived artifacts synchronized;
- residual risks and inspection gaps documented;
- reusable engineering knowledge classified.

Otherwise use `reporting-only`; keep the task blocked or incomplete.
Operational logs belong in `memory_bank/`. Promote only reusable, evidence-backed,
secret-free knowledge to `docs/engineering-memory/`.

## 17. Runtime Adapters and Compatibility

Existing `.codex/`, `.claude/`, MCP, plugin, OpenCode, and file-handoff layers
are adapters. Prefer native or official integrations when they satisfy governance.
Retain file-based handoff for durable queues, cross-machine work, recovery, or
formal audit requirements.

No adapter may redefine core authority, SSOT, evaluation rules, Hard Stops, or closeout.

---
> Source: [oleyna80/hardwarelab](https://github.com/oleyna80/hardwarelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
