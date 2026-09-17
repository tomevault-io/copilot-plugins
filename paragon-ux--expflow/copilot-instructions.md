## expflow

> This file is the repository router and cross-cutting governance layer for the active Expflow Build Week.

# AGENTS.md

## 1. Purpose

This file is the repository router and cross-cutting governance layer for the active Expflow Build Week.

It is intentionally compact. Detailed phase order, review cadence, merge rules, evidence requirements, and gate criteria belong in:

```text
docs/internal/BUILD_WEEK_WORKFLOW_CURRENT.md
```

Detailed implementation requirements belong in the active phase prompt.

The repository-local precision review procedure belongs in:

```text
.agents/skills/expflow-build-week-pr-review-precision/SKILL.md
```

Do not duplicate those documents here.

---

## 2. Source-of-truth order

Read and obey sources in this order:

1. root and applicable nested `AGENTS.md`;
2. immutable architecture, protocols, schemas, registries, canonical examples, compatibility contracts, and protected-surface declarations;
3. `docs/internal/BUILD_WEEK_WORKFLOW_CURRENT.md`; <!-- config-docref -->
4. `docs/internal/CURRENT_STATUS_MATRIX.md`; <!-- config-docref -->
5. `docs/internal/GLOSSARY.md`; <!-- config-docref -->
6. the active phase prompt under `docs/internal/phase_prompts/`;
7. repository-local skills required by the active phase;
8. accepted phase and gate reports;
9. relevant source, tests, CI, and package scripts;
10. external reviews and historical evidence as findings to reproduce;
11. conversation context as non-canonical working context.

A lower source may clarify a higher source. It must not override it.

When controlling sources conflict materially, stop the affected work and record the contradiction. Do not select whichever interpretation is easiest to implement.

`docs/releases/v1.1.0/` is frozen release provenance. It is not active implementation authority.

---

## 3. Active workflow

The active Build Week sequence is:

```text
BW-A
  Phase 1 — Ordinary UX/UI Corrections
  Phase 2 — Expflow GUI Foundation

BW-B
  Phase 3 — Stable Read Models
  Phase 4 — Evidence Intake and Authority Reconciliation
  Phase 5 — Portable Workflow Package

BW-C
  Phase 6 — Evidence-Backed Gap Closure
  Phase 7 — Pilot and Empirical Evaluation

BW-D
  Phase 8 — Guerilla Profile and Event Contracts
  Phase 9 — Guerilla Causal Event View GUI
```

Phase 1 is inherited as implemented but remains subject to the current repository status and accepted review record.

The current phase is determined by the workflow and repository evidence, not by an external launcher, conversation, branch name, or agent assertion.

No later phase may begin before the preceding phase is accepted and merged according to the workflow.

---

## 4. Product boundaries

### 4.1 Native authority

Native systems remain authoritative for native state.

- the filesystem is authoritative for current bytes within its scope;
- Git is authoritative for Git history and references;
- Reqtrace is authoritative for resolved traceability state;
- FIMP is authoritative for its mutation transaction and receipt;
- Expflow is authoritative for Expflow records and projections under its contracts;
- Guerilla is authoritative only for its event records and supported causal assertions.

Observation does not transfer authority.

Guerilla must not repair, replace, or silently reinterpret Expflow-native state.

### 4.2 Ordinary CLI

The ordinary Expflow command set remains:

```text
init
sync
status
restore
```

A fifth ordinary command requires an explicit architecture and compatibility decision.

Locked exit behavior:

- uninitialized `status`: exit `0`;
- operational mutation failure: exit `1`;
- usage failure and unknown command: exit `2`.

### 4.3 Restore

Restore remains:

- byte-exact;
- append-only;
- forward-committing;
- recoverable;
- non-destructive to recorded history;
- non-mutating during preview;
- refusing conflicting unrecorded drift by default;
- explicitly overridable only through the approved contract.

### 4.4 Identity

A provisional identity must be labeled provisional wherever it is exposed.

A provisional identifier must not be described as committed, stable, accepted, or durable.

### 4.5 GUI

The product name is **Expflow GUI**.

The approved root is:

```text
apps/gui/
```

The GUI must use documented application operations and read models. It must not treat raw undocumented `.expflow` storage as an application contract or create GUI-owned canonical state.

### 4.6 Compatibility

Do not silently break:

- v1 command automation;
- machine-readable output;
- persisted records;
- package exports;
- recovery behavior;
- public extension behavior;
- frozen release evidence.

Breaking changes require an explicit architecture and versioning decision.

---

## 5. Status and claims

Use these dimensions separately:

- implemented;
- internally verified;
- ordinary-surface available;
- phase accepted;
- gate accepted;
- pilot verified;
- empirically evaluated;
- production supported.

Do not replace them with a maturity percentage.

Do not claim:

- implementation from a schema, plan, prompt, mock, or screenshot;
- usability from internal tests alone;
- empirical success from reviewer approval;
- ordinary-surface availability from a library export;
- causal certainty from correlation;
- completion from material output alone.

Unknown, pending, partial, conflicted, rejected, and failed states remain distinct.

---

## 6. Repository-only operation

Active execution must use repository-owned paths, controls, tests, reports, and skills.

Do not require:

- `Expflow-Test/` or `Expflow-Test-*`;
- a nested Build Week package;
- a user-home skill;
- an external harness;
- a machine-local review sandbox;
- an external chatbot.

Historical reports may retain old paths as factual provenance. Active instructions must use repository roles and paths.

An external launcher may orient an agent, but it is not authority and must not be required for execution.

---

## 7. Pass start

At the start of a phase or recovery pass:

1. inspect Git branch and working-tree state;
2. read the orientation index and applicable System 1/System 2 decisions;
3. determine whether a new ADR is actually required;
4. read this file and the active workflow;
5. read only the active phase prompt and relevant normative sources;
6. resolve required paths through repository configuration;
7. inspect relevant source, tests, and accepted reports;
8. record the integration base, phase branch, and runtime versions;
9. run the workflow-required baseline controls.

Do not reread the entire repository on every pass.

Create or revise an ADR only when a durable architecture, policy, or repeated-process decision is new or has materially changed. Packaging renames, path hygiene, ordinary report corrections, and one-off execution details do not require an ADR.

---

## 8. Implementation discipline

The parent orchestrator owns implementation.

For each bounded workstream:

- establish a concrete requirement;
- inspect the existing implementation;
- reproduce a gap when applicable;
- make the smallest architecture-consistent change;
- add focused tests;
- run focused checks;
- inspect the diff;
- checkpoint coherent work.

Do not:

- launch recursive agent hierarchies;
- allow a reviewer to implement;
- implement later-phase placeholders;
- broaden scope to consume remaining context;
- repeatedly rerun unchanged expensive checks;
- weaken tests or governance to obtain a pass;
- modify user-owned unrelated work.

A subagent may implement only a genuinely isolated workstream with an explicit file and behavioral boundary. The parent orchestrator must review and integrate its output.

---

## 9. Validation cadence

Use the narrowest sufficient check first.

Required expensive validation cadence is defined by the workflow. As a repository rule:

- run baseline full validation once at phase start unless the inherited integration commit already has current accepted evidence and no environment-sensitive dependency changed;
- run focused checks during implementation;
- run full validation for the phase candidate;
- rerun full validation after remediation only when production code, contracts, package boundaries, or test infrastructure changed;
- use targeted documentation, reference, formatting, and package checks for evidence-only corrections;
- run post-merge validation once.

Do not rerun the full suite merely because report prose changed.

A transient infrastructure failure may be rerun once using the identical command. Record both attempts. Repeated failure is a blocker, not permission to relabel the result.

---

## 10. Review governance

Every phase receives one comprehensive independent precision review using:

```text
.agents/skills/expflow-build-week-pr-review-precision/SKILL.md
```

That review creates a frozen finding ledger.

Closure review is bounded to:

- the frozen finding IDs;
- the remediation diff;
- regressions directly introduced by remediation;
- the evidence necessary to close those findings.

Closure review must not restart a general phase search.

New findings during closure are allowed only when directly introduced by remediation or deterministically exposed as the same root cause on another supported path.

Gate-closing phases receive one additional aggregate gate review:

- Phase 2: BW-A;
- Phase 5: BW-B;
- Phase 7: BW-C;
- Phase 9: BW-D.

Gate review is broader than phase review but does not repeat complete line-by-line phase review without a gate-level trigger.

Reviewer output is evidence. The parent orchestrator owns remediation and merge decisions under the workflow.

---

## 11. Evidence and reports

Store durable phase and gate evidence under repository-owned internal report paths.

A phase report must identify:

- exact base and head;
- scope delivered;
- finding dispositions;
- focused checks;
- full validation when required;
- package verification when required;
- protected-surface and compatibility status;
- known limitations;
- reviewer verdict;
- merge or next-action state.

Report corrections that only align commit references, status wording, formatting, or tables are administrative closeout unless they change a material claim.

Administrative closeout does not require a new full phase review. It requires deterministic parent verification and applicable documentation controls.

A reviewer report must not be rewritten to make its verdict appear stronger.

---

## 12. Protected and configured surfaces

Marked configuration references are incomplete until:

```text
npm run check:config-references
```

passes against the required staged snapshot.

Repository-local skill changes are incomplete until:

```text
npm run check:skill-contracts
```

passes.

Protected architecture and frozen release bodies are controlled separately through:

```text
npm run check:protected-surfaces
```

Do not bypass, weaken, or merge these controls into one ambiguous check.

Generated caches, bytecode, package metadata, scratch output, and local process traces must remain untracked unless a normative golden artifact explicitly requires them.

---

## 13. Merge policy

Keep `main` protected.

Use a rolling non-protected Build Week integration branch and one branch per phase.

A phase may merge only when the workflow’s acceptance conditions are satisfied.

The exact reviewed head must be the exact accepted phase head. Any production-code change after review invalidates that acceptance and requires bounded re-review.

Documentation-only administrative closeout after PASS may be included before merge when:

- it does not change behavioral or gate claims;
- deterministic checks pass;
- the accepted reviewed code head remains recorded;
- the workflow permits it.

Do not merge directly to `main` without separate user authorization.

---

## 14. Stop conditions

Stop the affected work when:

- authority conflicts materially;
- phase scope is ambiguous;
- required repository-local controls are missing;
- a protected surface would require unauthorized change;
- compatibility cannot be preserved without a decision;
- working-tree ownership is ambiguous;
- a verified finding requires unauthorized scope;
- validation cannot be made trustworthy;
- the review skill cannot be applied independently;
- the reviewed head changed materially before merge;
- context is insufficient for a deterministic handoff.

A stop handoff must record exact Git state, completed work, last passing check, blocker, and next bounded action.

---

## 15. Final principle

Prefer fewer, stronger control points.

One phase review should discover phase defects.

Focused closure should prove those defects resolved.

One gate review should assess cross-phase readiness.

Do not turn every correction into a new full review, every report edit into a full validation cycle, or every execution inconvenience into an ADR.

<!-- config-reference-index:start -->

- `.config-reference-reconciliation.yaml` - repository governance role

<!-- config-reference-index:end -->

---
> Source: [paragon-ux/Expflow](https://github.com/paragon-ux/Expflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
