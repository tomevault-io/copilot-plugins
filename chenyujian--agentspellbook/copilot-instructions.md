## agentspellbook

> > **Top rule:** Never accept a healthier user-facing state than the persisted

# AgentSpellbook Development Agent Instructions

> **Top rule:** Never accept a healthier user-facing state than the persisted
> evidence supports. When uncertain or partially failed, report
> degraded/stale rather than fresh/complete.

> **Compliance rule:** When existing code, tests, documentation, or checklists
> violate this file, correcting that violation takes priority over starting the
> next milestone. Adding a rule does not resolve the defect that motivated it.

## Project context

Repository: `$REPO` (the local AgentSpellbook checkout)
Rust workspace, edition 2024, toolchain pinned at 1.96.1.
Hexagonal architecture: domain <- application <- transports (CLI/MCP).
Application crate must not depend on SQLite, Clap, MCP, tokio, regex, or
platform crates.

## How to read current state

`AGENTS.md` intentionally avoids hardcoded HEAD SHAs, test counts, and
milestone completion status because they expire between sessions. Read the
live state with commands instead:

```bash
git log --oneline -5                                  # recent commits
git rev-parse HEAD                                    # current commit
cargo test --workspace --all-targets 2>&1 | grep 'test result:'  # test count
git status --short                                    # working tree
```

Dated snapshots (commit SHA, toolchain, schema version, test count,
release-binary hash, provider fingerprints) live in
[docs/release/m0-baseline-capture.md](docs/release/m0-baseline-capture.md).
Milestone task status lives in
[docs/release/m0-release-checklist.md](docs/release/m0-release-checklist.md)
and is the single source of truth for what is done.

### Trial readiness plan

7 engineering days (M0-M5) + 14-day user trial. The full plan with complete
task IDs, exit gates, and trial success criteria lives in
[docs/release/user-trial-readiness-plan.md](docs/release/user-trial-readiness-plan.md).

- **Feature freeze in effect (M0-05):** no new feature work is accepted until
  M5 is complete. Only milestone tasks in the trial readiness plan proceed.
  Any out-of-scope request requires a separate product decision.
- Check the [release checklist](docs/release/m0-release-checklist.md) for
  current milestone status rather than relying on this file.

## Mandatory defect-closure protocol

Passing the existing test suite is not the same as passing acceptance.

A correctness, persistence, privacy, freshness, cursor, migration, or transport
defect is closed only when all five closure artifacts exist:

1. **Production fix**
   - The root cause is fixed, not only the reviewed line or visible symptom.

2. **Same-root audit**
   - Search the affected data flow for the same unsafe pattern.
   - Inspect inputs, persistence, public response, CLI, MCP, restart behavior,
     and aggregate state where applicable.
   - Classify every match as fixed, explicitly justified, or deferred with an
     owner and milestone.

3. **Negative regression test**
   - Reproduce the pre-fix failure.
   - Exercise the real public binary or real MCP JSON-RPC transport.
   - Inject the failure rather than manually writing the expected final state.
   - Assert structured JSON fields, exit code, persisted state, and behavior
     after process restart.
   - Privacy-sensitive changes must use secret, Unix-path, Unicode-path,
     Windows-path, and UNC canaries.

4. **Claim reconciliation**
   - Update checklist items, test names, evidence documents, and milestone
     status to describe what the tests actually execute.
   - A test that writes state through the Store API is not a real sync test.
   - A binary used only for retrieval does not prove that the binary performed
     the preceding state transition.

5. **Committed evidence**
   - The implementation, tests, and updated claims must all be tracked and
     committed before reporting completion.

If any closure artifact is missing, report `IN PROGRESS`.

### Outcome-first execution

Before editing, write one outcome sentence:

> Make [user-visible state] truthful across [real operation] and [restart or
> transport boundary] under both success and injected failure.

Then define:

- in-scope files and surfaces;
- failure scenarios;
- required public-path evidence;
- explicit non-goals;
- COMPLETE versus IN PROGRESS conditions.

Do not begin implementation until this outcome contract is written.

### Full-path audit

For runtime state and diagnostic changes, inspect the complete path:

```
source/provider
-> discovery/probe/ingestion
-> cursor and state persistence
-> aggregation
-> public response
-> CLI
-> MCP
-> process restart
```

A fix at one stage is incomplete if a later stage can leak, suppress,
contradict, or fabricate the same information.

### Correctness-change test rule

A production change involving persistence, privacy, freshness, cursor,
migration, state transition, or error propagation must add or materially
strengthen a regression test for the failing path.

An unchanged passing test count is not evidence that the newly changed failure
path is covered. If an existing test already covers it, the completion report
must identify the exact assertions that failed before the fix and pass after
it.

### No literal-finding fixes

Review findings are examples of violated contracts, not an exhaustive patch
list.

Do not fix only the exact line cited by a reviewer. Trace the root cause,
search sibling occurrences, and close the full outcome contract.

Do not expand into unrelated milestones or features.

### Acceptance report template

Every implementation report must contain:

1. **Outcome**
   - The original outcome sentence.
   - `COMPLETE` or `IN PROGRESS`.

2. **Implemented behavior**
   - Production behavior changed.

3. **Same-root audit**
   - Search commands.
   - Every relevant match and its classification.

4. **Public-path evidence**
   - Exact command.
   - Fixture or injected failure.
   - Exit code.
   - Structured assertions.
   - Raw output location.
   - Process-restart result.
   - CLI/MCP result where applicable.

5. **Privacy evidence**
   - Canaries used.
   - stdout leaks: 0
   - stderr leaks: 0
   - persisted leaks: 0
   - MCP leaks: 0

6. **Remaining violations**
   - Any unresolved correctness conflict forces `IN PROGRESS`.

7. **Repository state**
   - HEAD SHA.
   - Tracked/untracked state.
   - Files included in the commit.

8. **Gate results**
   - fmt
   - clippy
   - full tests
   - ignored count
   - diff-check

A passing gate sequence cannot override missing public-path acceptance evidence.

## Mandatory workflow (read before every coding session)

1. Run `git status --short` and `git log --oneline -3` to understand
   the current state.
2. Read `docs/evaluation/` for the latest acceptance findings.
3. Before editing any function, `grep` all call sites and test
   assertions that reference it.
4. After editing, run the full gate sequence (see below) before
   claiming completion.

## Engineering principles

### 1. Preserve module cohesion

Place code according to ownership and cohesion.

- Extend an existing module when the behavior belongs to its established
  responsibility.
- Create a new module when the behavior has a distinct responsibility,
  compatibility boundary, or independently testable contract.
- Do not create a new file solely to avoid understanding or safely modifying
  existing code.
- Before creating a module, identify its owner, public API, callers, and tests.

### 2. Clear all `#[ignore]` tests

Every `#[ignore]` is a technical debt marker. If this session touches
the relevant code area, fix the ignored test. Do not leave new
`#[ignore]` annotations. Target: 0 ignored tests.

### 3. Verify against real data, not just synthetic tests

After each fix, check output against at least one real Codex/ZCode
session via the CLI:
```bash
./target/release/agentspellbook handoff <session_id>
```
Synthetic tests pass; real sessions reveal edge cases.

### 4. Read full context before editing

- `grep -rn "function_name"` to find all call sites.
- Check serde attributes on structs before changing fields.
- Check test assertions that depend on constants you are changing.
- 30 seconds of grep saves 10 minutes of debug.

### 5. Investigate related defects without expanding scope silently

When fixing a defect, search for the same unsafe pattern in the affected
subsystem.

- Fix related occurrences that share the same root cause and remain within the
  authorized task.
- Record unrelated occurrences as separate follow-up work.
- Do not silently expand into another milestone, provider, transport, or public
  contract.
- If fixing the defect requires a product decision or scope expansion, stop and
  request direction.

### 6. Keep infrastructure work proportional and in scope

Infrastructure changes are justified when they are required to implement,
verify, or safely release the current task.

Prioritize:

- regression tests for the changed behavior;
- CI coverage required by the acceptance gate;
- deterministic fixtures;
- documentation directly affected by the change.

Do not perform unrelated refactoring, dependency upgrades, CI expansion, or
documentation cleanup merely to satisfy a fixed time percentage. Record such
work separately.

### 7. Clippy discipline

Write code with common lints in mind:
- `&mut` vs `&` borrow patterns.
- `map_or` vs `map().unwrap_or()`.
- `too_many_lines` (split or `#[allow]`).
- `needless_collect`, `redundant_clone`.
- `let...else` instead of `match ... None => return`.
Run `cargo build` before `cargo clippy` before `cargo test`.

## Correctness and acceptance rules

The defect-closure protocol above defines the mandatory workflow for closing
defects. The rules below add project-specific invariants that are not fully
covered by the general protocol.

### Never silently discard correctness-critical errors

- Do not use `let _ = ...`, `.ok()`, `unwrap_or_default()`, or equivalent
  suppression for persistence, migration, cursor, lock, privacy, or freshness
  operations unless fail-open behavior is explicitly part of the contract.
- Fail-open applies only to **transient unavailability** (a momentarily
  unreadable table, a lock probe) and **legitimate absence** (a clean index, a
  deleted row). It never applies to **corruption**. These three states are
  distinct and must produce distinct user-facing results:
  - corrupt data -> an explicit error with a stable `STORE_CORRUPT` code;
  - unreadable/missing infrastructure (e.g. a dropped table) -> an explicit
    error with a stable `STORE_UNAVAILABLE` code;
  - legitimate absence (clean index, empty read) -> the conservative default
    (`not_started` / stale), exit 0.
  Collapsing corruption into the legitimate-absence path (e.g. a lossy parser
  that maps an unknown enum to `NotStarted`, or `map_or_else(|_| default)`)
  is a defect even when the comment says "fail-open".
- A new user-visible failure class requires a corresponding stable
  `ErrorCode` variant. Do not encode a new failure as an existing code plus a
  string prefix; the transport boundary must be able to surface a
  machine-readable code and correct `Retryable` semantics. Do not declare a
  correctness-relevant error-contract change a non-goal to avoid the work.
- If an error is intentionally suppressed, add:
  - the reason;
  - the observable fallback behavior;
  - a structured diagnostic;
  - a regression test.
- A successful user-facing operation must not be returned when required state
  persistence failed.
- Check foreign-key and ordering requirements before writing dependent rows.
  In particular, state referencing a source cannot be persisted before the
  source itself exists.

### Preserve transaction invariants

- Canonical upserts, cursor advancement, and authoritative sync-state updates
  must be atomic when they describe one successful sync operation.
- Crash-safety has two layers; both are mandatory at M1 unless the second is
  explicitly deferred with an owner:
  1. **Pessimistic marker (M1):** before any correctness-sensitive provider
     work, persist a non-healthy marker (`InProgress` / `Failed`) so a crash
     or a later failed terminal write cannot leave a prior `Complete` / fresh
     state standing. This does not depend on single-transaction atomicity.
  2. **Single-transaction atomicity (M2-06):** fold the marker, the upsert,
     the cursor, and the terminal state write into one transaction.
  Do not use the M2-06 deferral to skip the M1 marker.
- Do not mark a transactional requirement complete while a related TODO
  explicitly defers transaction integration.
- Add interruption tests at each transaction boundary.
- After interruption, retry must converge without:
  - skipped source records;
  - duplicated canonical events;
  - advanced cursors with missing data;
  - fresh status backed by stale state.

### Aggregation rules require truth-table tests

Before implementing aggregation logic, write a truth table covering every
combination of:

- bootstrap state;
- compatibility state;
- source availability;
- prior successful sync;
- optional versus required source;
- single-source versus multi-source provider.

Each row must specify:

- provider status;
- global bootstrap state;
- `is_stale`;
- retained timestamps.

Tests must include every enum variant. A variant must not fall through to a
healthier status merely because it was omitted from a condition.

### State-transition requirements

For every persisted state machine, test the following transitions where
applicable:

- clean -> not started
- not started -> in progress
- in progress -> complete
- complete -> failed while retaining last success
- failed -> recovered
- complete -> degraded
- partial -> complete
- source unavailable before first successful sync
- provider removed
- index deleted and rebuilt
- process interrupted between writes

Assertions must verify:

- the public response;
- the persisted database state;
- behavior after process restart;
- agreement between aggregate state and per-provider/per-source state.

A response is incorrect if its fields contradict one another, for example
`compat_status=degraded` with `provider.status=fresh`.

### Keep documentation synchronized

Before completing a milestone, check and update together:

- `AGENTS.md`
- the milestone checklist
- compatibility matrix
- schema/version documentation
- release notes
- reproducible commands

Search for stale values with:

```bash
rg -n "SCHEMA_VERSION|schema version|tests|passed|DONE|complete|HEAD:" \
  AGENTS.md docs README.md
```

## Pre-milestone compliance audit

Before starting the next milestone, inspect the current repository for known
violations of this file.

Run:

```bash
git status --short
git log --oneline -5

rg -n \
  'let _ =|\.ok\(\)|unwrap_or_default\(\)|from_str_lossy|_lossy|TODO|FIXME|\[x\].*(transaction|CLI|MCP)' \
  crates docs AGENTS.md
```

For every match in a correctness-critical path, classify it as:

- compliant and explicitly justified;
- accepted limitation with a documented owner and milestone;
- defect that must be fixed before proceeding.

The audit must also compare:

- checklist completion claims against implementation;
- test names against the surfaces actually executed;
- compatibility status against user-facing freshness;
- schema constants against compatibility and release documentation;
- newly introduced rules against existing production code.

If a P1 violation contradicts the objective of the current milestone:

- reopen the milestone;
- change its checklist status from complete to in progress;
- fix the violation;
- add a regression test through the real public path;
- rerun the complete gate;
- only then start the next milestone.

Do not treat documenting a known violation as equivalent to fixing it.

## Gate sequence (must all pass before commit)

```bash
cd "$REPO"
export PATH="$HOME/.cargo/bin:$PATH"

cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace --all-targets
git diff --check
```

Acceptance: failed=0, ignored=0, fmt exit=0, clippy exit=0,
diff-check exit=0.

## ASB-HO-001 harness (when available)

```bash
cargo run --manifest-path /tmp/agentspellbook-eval/Cargo.toml
```

Verify: objective correct, decisions 2/2, completion verified then
contradicted, unresolved 2/2, next step actionable, privacy 0 leaks.

## Redaction rules (do not regress)

- Shell-quoted: `key="value"`, `key='value'`
- JSON: `"key":"value"`, `"key": "value"`
- Unlabeled: `sk-`, `ghp_`, `ark-`, `github_pat_`, `xoxb-`, `xoxp-`
- Bearer: `Authorization: Bearer <value>`, standalone `Bearer <value>`
- Paths: Unix absolute, Windows drive, UNC, Unicode paths
- Idempotent: second pass produces zero new replacements
- Unicode-safe: char-based scanning, never slice a lowercased copy
- Error-rendering discipline: an error type's `Display`/`fmt` implementation
  renders only the stable code, never the human message. Envelope messages
  can carry user input or internal error text; render them only at the
  transport boundary through an explicit sanitizer (e.g.
  `AppError::safe_external_message` -> `redact_sync_message`). A bare
  `{e}` / `e.to_string()` at the CLI/MCP boundary is a defect even when the
  message *usually* reads clean, because the next field change can leak.

## Claim extraction rules (do not regress)

- Objective: latest active task, not first user message; skip context
  injections via known wrapper tag allow-list (not `<` prefix).
- Decisions: action verbs only (chose/configured/decided); exclude
  questions, negations, teaching, hypotheticals, rule text, Markdown
  headers, definitions.
- CompletedWork: requires completion verb + `is_completed_work` gate;
  verified (ToolResult) = SourceExplicit, reported (no ToolResult) =
  Extracted; superseded_by references real claim ID.
- Unresolved: TODO/FIXME/still-fails + trailing ToolCall; exclude
  capability descriptions, future plans, teaching.
- Supersession: topic-matched (shared keyword or objective bridge);
  no cross-topic fallback.
- Active task: all extractors use `active_events()` slice.

## Commit conventions

- DCO sign-off: `git commit -s`
- English commit messages.
- One logical change per commit.
- Include before/after verification in the commit body.
- Do not commit unless the user asks.

---
> Source: [chenyujian/agentspellbook](https://github.com/chenyujian/agentspellbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
