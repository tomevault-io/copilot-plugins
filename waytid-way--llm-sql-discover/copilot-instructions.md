## llm-sql-discover

> Build the V1 Fresh-Context SQL Discovery & Trace pipeline defined by the normative specification, amendment, Epic #27, and approved governance documents. V1 discovers and traces source behavior and produces MySQL/PostgreSQL conversion projections. It does not execute APIs or databases and does not rewrite source code.

# AGENTS.md

## Project mission

Build the V1 Fresh-Context SQL Discovery & Trace pipeline defined by the normative specification, amendment, Epic #27, and approved governance documents. V1 discovers and traces source behavior and produces MySQL/PostgreSQL conversion projections. It does not execute APIs or databases and does not rewrite source code.

## Read before any work

1. `docs/superpowers/specs/2026-07-29-fresh-context-sql-discovery-trace-design.md`
2. `docs/superpowers/specs/2026-07-29-fresh-context-sql-discovery-trace-amendment-v1.1.md`
3. `docs/superpowers/specs/2026-07-29-epic-anchored-incremental-decomposition-design.md`
4. `docs/superpowers/plans/2026-07-29-llm-sql-discover-implementation-plan.md`
5. `docs/superpowers/plans/2026-07-29-file-boundary-execution-policy.md`
6. `docs/superpowers/plans/llm-sql-discover/file-touch-map.json`
7. Epic `#27`, the current Round record, and the assigned Issue including comments and dependencies

The Architecture and Contract Specification and its approved amendments are normative. An Issue, prompt, implementation detail, or discovery may not silently change contract semantics or the V1 Definition of Done.

## Epic-anchored incremental workflow

- Epic `#27` is the permanent V1 anchor.
- Implementation is planned in approved Rounds anchored to an exact verified `main` SHA.
- A Round contains one to three Issues, but only one implementation Issue may be active or queued for merge.
- Future-Round Issues are not created until the current Round passes peer review, sequential merge, post-merge smoke verification, and discovery-ledger triage.
- Archived Issues `#12`–`#26` are planning history and must never be selected for implementation.
- The implementation plan is an Architecture Horizon for future capability, not permission to implement archived task bodies verbatim.

## Current Round

- Round ID: `R1`
- Planning anchor SHA: `9275a218fc30bdcab278c027a704a7541b8a5447`
- Parent Epic: `#27`
- Sole planned Issue: `#11`
- Native parent status: must be verified before implementation. A Markdown reference is not equivalent to a native sub-issue relationship.
- Round R1 completes only after #11 receives the final-Round PASS verdict, merges, and passes its post-merge smoke gate.

## Issue eligibility gate

Before starting work, the Coding Agent must verify all of the following:

1. The Issue belongs to the current approved Round.
2. The Issue is a verified native sub-issue of Epic `#27`, unless an explicit human-approved temporary exception is recorded in the Epic.
3. The Issue's `planning_anchor_sha` is an ancestor of current `main`.
4. Every `blocked-by` dependency is closed and merged.
5. No other implementation Issue or PR is active or queued for merge.
6. The Issue has a complete execution contract and exclusive file write sets.
7. The Issue is not archived, superseded, horizon-only, or part of an unapproved future Round.

Failure of any gate blocks implementation. Do not infer that an open Issue is executable.

## Round transition gate

Do not plan or create the next Round until:

- every current-Round Issue has independent Senior Expert Peer Review PASS;
- every approved PR has merged sequentially;
- post-merge smoke verification passes on `main`;
- the resulting remote `main` SHA is recorded;
- no Critical or Important finding remains;
- no implementation PR remains open or queued;
- the Discovery Ledger is triaged;
- the fixed V1 Definition of Done is unchanged or has an approved amendment;
- governance documents and file ownership maps are consistent.

## File-boundary and single-writer rules

- The Issue's exclusive production write set is the complete set of production files it may create or modify.
- Test files require a separate declared test write set.
- Files outside declared write sets are read-only.
- A required undeclared edit stops the task. Amend the Issue and Round file map before coding.
- Never use broad staging such as `git add -A`; stage explicit authorized paths.
- A shared or hotspot file has one owner at a time.
- `src/sqltrace/cli.py` is owned by Round R1 Issue #11 and becomes a stable registry shell. Later commands must be added through isolated command modules.
- Configuration must be split by subsystem; do not create a generic shared `config.py` hotspot.
- Root governance files change only through an approved governance migration.

## Required Issue contract

Every executable Issue must state:

- Parent Epic and verified relationship status;
- Round ID and planning anchor SHA;
- capability slice, predecessor, and native blocked-by dependencies;
- merge/start gate;
- exclusive production and test write sets;
- read-only dependencies and frozen hotspots;
- normative Contract IDs;
- input validation rules;
- error and state-transition behavior;
- timeout, cancellation, retry, and idempotency policy;
- focused tests, regression tests, and exact verification commands;
- fixture and coverage expectations;
- semantic-review checklist;
- discovery handling rules;
- post-merge smoke gate;
- commit boundary, non-goals, and forbidden edits.

An Issue missing these fields is not executable.

## Discovery policy

Classify discoveries before creating work:

- **Required in-scope correction:** implement within the current Issue when required by its acceptance criteria and inside its write set.
- **Out-of-scope blocker:** stop, record it in the Epic Discovery Ledger, obtain human triage, and create or reorder a sub-issue only after approval.
- **Non-blocking discovery:** record it in the Discovery Ledger for the next Round; do not create an Issue immediately.
- **Normative discovery:** stop planning and require an approved specification or Definition-of-Done amendment.

## Architecture invariants

- Static analyzers own source coordinates and Evidence Anchors.
- One LLM request contains one Analysis Unit from one file and no rolling history.
- C# analysis is semantic-first with fact-level degradation.
- Vue parser-local offsets must round-trip to original snapshot bytes before persistence.
- SQLite state and outbox rows commit atomically.
- LLM output is advisory and non-authoritative until schema, identity, anchor, ownership, and invariant validation pass.
- Resolver confidence cannot override missing, ambiguous, or conflicting deterministic proof.
- SQL candidates must be promoted through normalization before conversion.
- MySQL and PostgreSQL projections are independently versioned and invalidated.
- Reports are read-only projections and must not trigger analyzer or provider calls.

## Testing and verification

Use TDD: observe a focused failing test, implement the minimum production-capable correction, pass the focused test, then pass the required regression gate.

Before requesting review run:

1. Every command listed in the Issue.
2. The owning Round/bundle regression suite.
3. Contract and registry checks when public contracts are involved.
4. `git diff --check`.
5. A changed-file ownership audit.
6. A clean-status check.

Do not claim completion from partial, previous, inferred, or agent-reported results.
## Reviewer handoff

- The Coding Agent MUST NOT spawn or dispatch an independent Peer Reviewer after implementation.
- After verification, the Coding Agent MUST provide a copyable review handoff report containing the exact repository, Issue, Round, base/head SHAs, changed-file ownership audit, TDD evidence, verification commands/results, acceptance matrix, semantic checklist, discoveries, risks, and known limits.
- The repository maintainer is responsible for forwarding that report to the independent Senior Expert Peer Reviewer.
- The independent Senior Expert Peer Review and its required verdict remain mandatory before merge. The Coding Agent MUST NOT claim review PASS, merge readiness, Round completion, or post-merge smoke success.

## Semantic peer review

Review must go beyond compile, lint, and test status. Check at minimum:

- source byte, line, Unicode-scalar, and UTF-16 coordinate correctness;
- stale snapshot or stage-result use;
- transaction, lease, crash, cancellation, retry, and race behavior;
- false promotion of candidate or ambiguous evidence to authoritative evidence;
- deduplication and idempotency semantics;
- cache fingerprints and invalidation boundaries;
- route, DI, DTO, request-body, parameter, and SQL lineage meaning;
- timezone, collation, case sensitivity, null semantics, and target-dialect assumptions;
- MySQL/PostgreSQL isolation;
- report provenance and accidental provider/analyzer calls;
- Round membership, planning-anchor ancestry, dependency relationships, and discovery classification.

Critical and Important findings block merge. The final Issue of a Round requires `PASS — READY FOR ROUND COMPLETION AND REASSESSMENT`.

## Coding conventions

- Python 3.12+, typed public APIs, Pydantic v2 contracts, SQLAlchemy 2/Alembic state.
- Keep files focused by responsibility. Avoid generic `utils.py` and cross-layer helper dumping grounds.
- Public names and contract fields must match the normative specification.
- Use deterministic IDs and canonical serialization where required.
- Keep stdout machine-readable for JSON/JSONL modes; progress and diagnostics go to stderr.
- Do not add V2 scope, API execution, database execution, source rewriting, or automatic implementation expansion.

## Commit identity

- New implementation commits MUST use committer identity `codex`.
- Before committing, the Coding Agent MUST verify the effective `user.name` and `user.email` for the commit and report them.
- Existing commits are immutable; this rule applies only to commits created after this governance change.
## Git and merge discipline

- Create a branch/worktree from current `main` only after eligibility gates pass.
- Rebase before final verification when `main` changes.
- Merge one implementation Issue at a time.
- The reviewed head SHA must not change after PASS without a new independent review.
- After merge, run the declared post-merge smoke gate on `main`.
- Do not start another Issue until the coordinator authorizes it.
- When the final Issue of a Round completes, enter Round Reassessment Mode; do not implement the next capability automatically.

---
> Source: [Waytid-way/llm-sql-discover](https://github.com/Waytid-way/llm-sql-discover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
