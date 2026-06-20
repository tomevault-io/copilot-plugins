## spec-tdd

> Run a spec-driven, test-first implementation workflow for software projects. Inserts three human review gates (architecture, patterns, tests) before any implementation, plus optional architecture discovery, SPEC creation, acceptance criteria, failing tests, frozen tests, and implementation loops until verification passes.


# Spec-TDD Implementation Workflow (with three review gates)

You are running a strict software-delivery workflow with **three mandatory human review gates** layered on top of test-first development:

```text
understand → architecture → [GATE 1] → patterns → [GATE 2] → specify → tests → [GATE 3] → freeze → implement → verify
```

The three altitudes mirror an industry pattern: humans set direction at the top (architecture), guide the middle (patterns/abstractions), and delegate file-level code to the agent. Each gate is enforced by hooks — you cannot bypass them by editing files manually.

User request / arguments:

```text
$ARGUMENTS
```

Respond in the user's language. Keep implementation artifacts in the repo language unless the user asks otherwise.

## Non-negotiable rules

1. **No production implementation before red tests AND all three gates approved.** You may create or repair test infrastructure first, but do not implement feature behavior until at least one relevant new/changed test fails for the correct reason AND `state.approvals.{architecture,patterns,tests}` are all set.
2. **Three review gates are mandatory.** After writing the architecture draft, after writing the patterns/abstractions draft, and after writing red tests, you MUST stop and present the artifact to the user. You must not proceed until the user replies `approved <gate>` (case-insensitive). Then you run `state-update.mjs --approve <gate>` to record the approval before continuing.
3. **Each gate must include a sub-agent review.** Before asking the user to approve, invoke an independent sub-agent (`general-purpose`) that re-reads inputs in a clean context and writes a review report. Attach the report when you ask the user for approval.
4. **SPEC, patterns, and tests are sources of truth.** Implementation follows `00-ARCHITECTURE.md`, `00b-PATTERNS.md`, `01-SPEC.md`, `02-ACCEPTANCE.md`, `03-TEST_PLAN.md`, and `04-TRACEABILITY.md` under `.ai/spec-tdd/<feature-slug>/`. Contracts in `src/contracts/` (if produced) are part of the patterns artifact and may not be silently rewritten during implementation.
5. **Freeze before implementation.** After red tests are proven AND tests gate is approved, run the freeze script. Frozen tests, acceptance criteria, traceability, contracts, and verify scripts must not be edited during implementation unless the user explicitly approves.
6. **Verification owns completion.** Completion requires the configured verify command to exit 0. Do not claim completion while verify fails.
7. **Fix root causes only.** Do not delete, skip, weaken, hardcode around, or mock away behavior just to pass tests.
8. **Prefer behavior tests through public interfaces.** Public surfaces are defined in `00b-PATTERNS.md`. Avoid testing private functions, internal call order, and implementation details. Mock only uncontrollable boundaries: network, clock, filesystem, platform APIs, payment providers, push notifications, external services.
9. **Slice large work.** For large projects, produce the full architecture + patterns + SPEC + test plan first, then implement one vertical slice at a time: one behavior group red → freeze → green → refactor → extend with the next slice. Each slice's tests still need a tests-review approval if SPEC/AC entries are added.
10. **Be honest about blocked verification.** If dependencies, credentials, emulator/simulator, DB, or browser drivers are unavailable, create the best local tests possible and report the exact blocked command and missing requirement. Do not pretend verification passed.

## Supporting files bundled with this skill

Load these only when needed:

- `templates/01-SPEC.template.md` — canonical SPEC structure.
- `templates/02-ACCEPTANCE.template.md` — acceptance checklist and Definition of Done.
- `templates/03-TEST_PLAN.template.md` — test strategy and red-test protocol.
- `templates/04-TRACEABILITY.template.md` — requirement ↔ test ↔ command matrix.
- `templates/05-TASKS.template.md` — vertical-slice task breakdown.
- `templates/00b-PATTERNS.template.md` — module map + ADRs + interface contracts.
- `templates/verify.sh.template` — portable verification script template.
- `reference/test-design.md` — test-quality rubric for web, mobile, desktop, backend, CLI, and libraries.
- `reference/spec-quality-rubric.md` — SPEC quality gates.
- `reference/platform-matrix.md` — platform-specific verification examples.
- `scripts/state-update.mjs` — manages `.ai/spec-tdd/state.json` (phase, approvals, slug, verify command).
- `scripts/freeze-tests.mjs` — records hashes of frozen tests and gate files (refuses to freeze if any gate is unapproved).
- `scripts/check-frozen-tests.mjs` — fails if frozen tests/gates changed or new skip/only markers appear.
- `scripts/guard-frozen-tests.mjs` — hook that blocks direct edits to frozen files during implementation.
- `scripts/guard-review-gates.mjs` — hook that blocks edits outside the review-phase whitelist when a gate is unapproved.
- `scripts/verify-on-stop.mjs` — Stop hook that keeps Claude working while verification fails during implementation phase.

## Workflow state

Use a feature slug such as `auth-login`, `checkout-flow`, or `desktop-sync`. Create:

```text
.ai/spec-tdd/<feature-slug>/
  00-ARCHITECTURE.md          # always required (Mermaid system diagram + materials log + constraints)
  00b-PATTERNS.md             # always required (module map + ADRs + contracts list)
  01-SPEC.md
  02-ACCEPTANCE.md
  03-TEST_PLAN.md
  04-TRACEABILITY.md
  05-TASKS.md
  06-IMPLEMENTATION_LOG.md
  frozen-tests.json           # created by freeze script
  reviews/
    architecture-review.md    # sub-agent review report for gate 1
    patterns-review.md        # sub-agent review report for gate 2
    tests-review.md           # sub-agent review report for gate 3
  logs/
.ai/spec-tdd/state.json       # active slug, phase, approvals, verify command, manifest path
src/contracts/                # optional; interface skeletons referenced by 00b-PATTERNS.md
scripts/verify.sh             # one command that proves the project is healthy
```

Phase values (state.json `phase`): `discovery`, `architecture_draft`, `architecture_review`, `patterns_draft`, `patterns_review`, `spec`, `test_design`, `red_tests`, `tests_review`, `implementation`, `extending`, `done`, `blocked`.

The approvals field has the shape `{ architecture: <ISO ts or null>, patterns: <ISO ts or null>, tests: <ISO ts or null> }`.

### Phase transition diagram

```text
discovery
   ↓
architecture_draft → architecture_review ──(approve)──→ patterns_draft
                                                           ↓
                                                   patterns_review ──(approve)──→ spec
                                                                                   ↓
                                                                              test_design
                                                                                   ↓
                                                                               red_tests
                                                                                   ↓
                                                                            tests_review ──(approve, no phase change)
                                                                                   ↓
                                                                           (run freeze) → implementation → done
                                                                                                            ↓
                                                                                                       extending
                                                                                                            ↓
                                                                                                  (re-approve where needed, re-freeze)
                                                                                                            ↓
                                                                                                    implementation
```

## Re-invocation: new feature vs. continuation

When this skill is invoked, **always check first** whether the user's request matches the current `active_slug` in `state.json`:

1. **Same feature, incomplete phase** — resume from the current phase. Do not restart. If you are on a review phase and approval is missing, your job is to re-present the draft to the user.
2. **Same feature, phase is `done` or `blocked`** — if the user wants to extend or fix it, follow the "Extending a completed or in-progress feature" procedure below.
3. **Different feature** — this is a new feature. Create a new slug, run `state-update.mjs --slug <new> --phase discovery` (or `architecture_draft` if `--skip-research`), and **start from Phase 0**. Do not skip to implementation. Do not reuse the previous slug's SPEC, tests, or frozen manifest.

**How to detect "different feature":** If the user's request describes capabilities, screens, modules, or behaviors that are not covered by the current slug's `01-SPEC.md`, it is a different feature. When in doubt, treat it as a new feature — skipping the gates is never acceptable for new work.

**Non-negotiable:** No production code for a new feature before its own architecture, patterns, and tests have all been approved, and its own freeze is executed.

**Slug switch is destructive:** When you call `state-update.mjs --slug <new>` with a slug that differs from the current `active_slug`, the script automatically **resets `approvals` to all-null, clears `manifest`, and rewinds `phase` to `discovery`**. The previous slug's prior state stays on disk (its frozen manifest, SPEC, tests are untouched) but is no longer the active state. Each feature carries its own gate history; do not assume a prior feature's approvals carry over.

### Extending a completed or in-progress feature (adding a slice)

When the user wants to add new behavior to an existing feature in `implementation`, `done`, or `blocked` phase:

1. **Verify existing tests still pass.** Run the verify command first. If it fails, fix existing implementation before extending.
2. **Transition phase to `extending`.** `state-update.mjs --phase extending`. Frozen tests remain hash-protected during extending.
3. **Decide which gates need re-approval.**
   - Pure additive change inside existing module boundaries: only **tests gate** needs re-approval. Set `approvals.tests = null` and go to `tests_review` after writing new red tests.
   - New module or changed module boundary: **patterns gate** must re-approve too. Set `approvals.patterns = null`, go through patterns_draft / patterns_review.
   - New external interface or system boundary: **all three** gates re-approve. Treat as new feature unless the user wants to keep history under same slug.
4. **Update SPEC and test plan.** Add new `R-*`, `AC-*`, `T-*` entries. Do not modify or remove existing ones.
5. **Write new tests and prove red.** New tests must fail for the right reason. Existing tests must still pass.
6. **Re-approve the relevant gates,** then re-freeze. `freeze-tests.mjs` will refuse if any required approval is missing.
7. **Implement and verify.** Normal Phase 6 loop.

### Unfreezing (when a frozen test or gate is genuinely wrong)

If during implementation (or just after freeze, before writing any production code) you discover a frozen test or gate document is wrong — a pinned literal output that's arithmetically inconsistent, an acceptance criterion that contradicts the architecture, a test asserting against the wrong contract — **STOP**. Do not implement against a wrong test. Do not write to a frozen file directly — the hook will block you. **Do not delete the manifest manually**: there is a proper command that creates an audit trail.

The correct procedure:

1. **Halt implementation and switch phase to `blocked`.** Run `state-update.mjs --phase blocked` immediately, *before* asking the user anything. The Stop hook only protects `phase = implementation`; while it sees implementation + failing verify, every turn-end is blocked and the agent looks like a runaway retry loop until Claude Code's Stop-hook safety cap fires. `blocked` quiets the Stop hook while you wait for the user. Do **not** issue further production-code edits in this phase.
2. **Explain the defect to the user.** Cite the specific frozen file and line, show the cross-check that proves the defect, propose the fix.
3. **Get user approval for unfreeze.** The user must reply with what they want changed.
4. **Run the unfreeze command:**
   ```bash
   node "$HOME/.claude/skills/spec-tdd/scripts/state-update.mjs" \
     --unfreeze \
     --reason "User approved: pinned value X is arithmetically inconsistent with cross-check Y; fixing to Z."
   ```
   This backs up the current manifest to `.ai/spec-tdd/<slug>/logs/unfreeze-<timestamp>.json`, appends an entry to `state.unfreeze_history`, and removes the active manifest so the hook stops protecting frozen files.
5. **Apply the fix across all affected files** — the test, the SPEC, the architecture worked examples, the traceability matrix — wherever the wrong value appears. The fix must be consistent across all artifacts.
6. **Decide gate re-approval scope** (same matrix as the `extending` procedure above):
   - Pin-value correction only, no contract change: re-freeze, no gate re-approval needed.
   - Acceptance criterion or test category changes: tests gate re-approval required (`state-update.mjs` with `--phase tests_review` then `--approve tests` after the user confirms).
   - SPEC requirement change: patterns gate re-approval required.
   - Architecture decision change: all three gates re-approve.
7. **Re-prove red** if any test was changed (vitest/your test runner must still fail because the new pinned contract isn't implemented yet).
8. **Re-freeze:** `node "$HOME/.claude/skills/spec-tdd/scripts/freeze-tests.mjs" ...`. The script refuses if approvals are missing. Re-freeze automatically advances phase back to `implementation`; the Stop hook resumes verify pressure on the next turn-end.

**Forbidden:** `rm`, `git rm`, `del`, `unlink`, manual deletion of `.ai/spec-tdd/<slug>/frozen-tests.json`, or any other bypass of the unfreeze command. The audit trail in `state.unfreeze_history` is the only legitimate record. Bypassing this means future readers cannot tell what was changed, when, or why — defeating the entire freeze mechanism.

## Phase 0 — Architecture discovery and material intake

Set phase: `node "$HOME/.claude/skills/spec-tdd/scripts/state-update.mjs" --slug <slug> --phase discovery` (or `architecture_draft` if `--skip-research`).

### Reading user-provided materials

When the user provides architecture documents, design docs, decision records, or any reference material:

1. **Read every provided file in full.** Use the Read tool on each file path. Do not skim, summarize from titles, or skip files because they "look similar." If a file is too large for a single read, read it in chunks until complete.
2. **Log what you read.** In `00-ARCHITECTURE.md`, list every file read with its path and a one-line summary. This creates a verifiable record.
3. **Extract all requirements, constraints, and decisions.** Every MUST/SHOULD/SHALL, every API contract, every data model, every scope boundary must be captured. Note conflicts between documents explicitly.
4. **Do not proceed to Phase 0.5 until all materials are fully read.**

### Discovery mode (no `--skip-research`)

Inspect the repo without writing implementation code. Identify project type, package managers, build tools, test frameworks, CI commands, architecture boundaries, public interfaces, persistence, external integrations, existing conventions, and risky areas (concurrency, auth, offline, migrations, security).

### Skip-research mode (`--skip-research`)

Skip repo exploration but still **read every provided document in full**. Synthesize into `00-ARCHITECTURE.md` and perform only a light consistency check (verify the project builds, test framework exists).

### Brownfield projects (existing codebase, discovery mode)

When the working directory already contains substantial source code (existing modules, tests, conventions), the workflow is **scoped to the new feature's blast radius**, not the whole system. Apply these rules during Phase 0:

1. **Architecture doc is feature-scoped, not system-scoped.** `00-ARCHITECTURE.md` documents:
   - A short overview of the existing system (1–2 paragraphs + a minimal Mermaid showing only the top-level components your feature will touch). Do **not** redraw the whole system.
   - **Existing modules to integrate with**: list specific files/modules the new feature will call into or extend, including the public APIs/interfaces you will consume verbatim. Cite file paths.
   - **Existing conventions to follow**: error handling, result types, logging, naming, DI, error code taxonomies. Cite where each convention lives.
   - **What's new**: the components, modules, types, files this feature introduces.
   - **Boundary of no-change**: explicitly state which existing modules/files you will NOT modify, so reviewers can verify the blast radius is bounded.
2. **Discovery budget is bounded.** Spend reads on: the modules your feature integrates with, the conventions you must follow, the risk areas (shared mutable state, migrations in progress, auth, performance budgets). Do not read files unrelated to your feature's surface area. A 100k-LOC repo does not need 100k LOC of reading.
3. **If the existing repo has a documented architecture (`docs/architecture.md`, `ARCHITECTURE.md`, `docs/adr/`), read it.** Cite the relevant decisions in your `00-ARCHITECTURE.md` and `00b-PATTERNS.md` — do not invent parallel decisions.
4. **If conventions conflict with what the user asked for, surface the conflict.** Architecture review is the place to settle "we use Result everywhere but the request implies throws".

### `00-ARCHITECTURE.md` requirements

This file is **required** (no longer optional). It must contain:

- **Materials read** (file list with summaries).
- **System diagram in Mermaid** (`flowchart` or `graph`): top-level components, data flow, external boundaries. This is the canonical picture the user will approve. In brownfield, this diagram shows only the **portion of the system your feature touches**, not the whole repo.
- **Facts, evidence paths, contracts extracted, assumptions, conflicts found, unknowns.**
- **System boundaries, data flow, high-level components, key constraints** — these correspond to "top altitude" items.
- **For brownfield**: also include the existing-modules-to-integrate-with list, the conventions-to-follow list, and the boundary-of-no-change list (per the four bullets above).

When the draft is complete, advance: `state-update.mjs --phase architecture_review`.

## Phase 0.5 — Architecture review gate (human-required)

When `state.phase = architecture_review` and `state.approvals.architecture` is null:

1. **Invoke a sub-agent for independent review.** Call `Agent` with:
   - `subagent_type: "general-purpose"`
   - `description: "Architecture coverage review"`
   - `prompt`: instruct it to read `00-ARCHITECTURE.md` and every source material file listed in its "Materials read" section, then write `.ai/spec-tdd/<slug>/reviews/architecture-review.md` with sections: **Coverage gaps**, **Conflicts with source materials**, **Hidden assumptions**, **Recommendations**. The sub-agent must read in a clean context — do not summarize prior conversation to it.
2. **Read the review report.** If it surfaces material gaps, revise `00-ARCHITECTURE.md` before presenting.
3. **Present to the user.** Show: (a) path to `00-ARCHITECTURE.md`, (b) the Mermaid diagram inline, (c) a 3-bullet summary of components and boundaries, (d) the sub-agent review report path and the top 3 findings.
4. **Ask explicitly:** "Reply `approved architecture` to proceed, or tell me what to change."
5. **On revision feedback:** edit `00-ARCHITECTURE.md` based on feedback, then loop back to step 1 (or skip sub-agent if changes are minor and you say so).
6. **On approval:** run `node "$HOME/.claude/skills/spec-tdd/scripts/state-update.mjs" --approve architecture`. Phase auto-advances to `patterns_draft`.

The `guard-review-gates.mjs` hook will block any Edit/Write/Bash that touches paths outside `.ai/spec-tdd/<slug>/` during this phase.

## Phase 1 — Patterns and abstractions

State should be `patterns_draft` (set by previous approval). If not: `state-update.mjs --phase patterns_draft`.

### Brownfield: reuse before invent

When the project already has patterns/contracts:

- **Existing ADRs / docs**: if `docs/adr/`, `ARCHITECTURE.md`, or similar exists, cite the relevant decisions in your `00b-PATTERNS.md` "Cross-cutting patterns" table with file references. Do not duplicate them as new ADRs.
- **Existing error/result types**: if the project already has a `Result` / `Either` / unified error class / `Logger` interface, **reuse it**. Reference the file in your patterns doc. Do not introduce a parallel mechanism.
- **Existing contracts**: if interfaces / types exist for surfaces your feature consumes, list them in the "Contract files" table with their existing paths. Only create **new** files under `src/contracts/` (or repo-appropriate path) for genuinely new public surfaces.
- **ADRs**: only write ADRs for **new non-obvious decisions** in your feature scope. "We use Result for errors" is not a new ADR if the project already uses it everywhere. If you have fewer than three new ADRs, that is fine — write what is real, not filler.
- **Departures from existing patterns**: if the new feature must depart from a pre-existing pattern, write an ADR explaining (a) why, (b) which module boundary protects the rest of the codebase, (c) the test that verifies the boundary holds.

### Required contents of `00b-PATTERNS.md`

Use `templates/00b-PATTERNS.template.md` to produce `.ai/spec-tdd/<slug>/00b-PATTERNS.md` containing:

1. **Module map.** A Mermaid diagram of modules/packages introduced or modified, with their direct dependencies. Existing modules your feature integrates with appear in the diagram but are visually distinct (e.g., dashed border, or a `subgraph existing`) from new ones. Cross-check: every component from `00-ARCHITECTURE.md` must map to one or more modules here.
2. **Module inventory table.** Each module: ID, name, path, one-sentence responsibility, public surface (exported classes/functions). For brownfield, mark each row as `Existing` or `New`.
3. **ADRs** for non-obvious design choices (error handling strategy, state management, concurrency, persistence, DI). At least one per genuinely new decision in this feature's scope; **for brownfield, fewer is OK if most decisions are inherited from the existing project** — but cite the inherited decision's source.
4. **Interface contracts.** Pick the language matching the project. Create skeleton files under `src/contracts/` (or repo-appropriate path) with empty types/signatures — **no bodies**. For brownfield, list both **reused** existing contracts (with existing file path; do not duplicate) and **new** contracts (new skeleton files). These contracts are committed and edited only via patterns review.
5. **Cross-cutting patterns table** (logging, error handling, time/clock, IDs, async). For brownfield, this is largely a citation of existing patterns; the table's "Enforcement" column references the existing mechanism. Only add new rows for patterns this feature introduces.
6. **Anti-patterns explicitly forbidden** table.

When draft is complete, advance: `state-update.mjs --phase patterns_review`.

## Phase 1.5 — Patterns review gate (human-required)

When `state.phase = patterns_review` and `state.approvals.patterns` is null:

1. **Invoke sub-agent.** `Agent` with `subagent_type: "general-purpose"`, instructing it to:
   - Read approved `00-ARCHITECTURE.md` and `00b-PATTERNS.md` and the skeleton files under `src/contracts/`.
   - Verify: every architecture component maps to a module; every module is testable through its public surface; ADRs cover non-obvious choices; contracts compile/type-check; patterns are enforceable.
   - Write `.ai/spec-tdd/<slug>/reviews/patterns-review.md` with: **Coverage gaps vs architecture**, **Testability concerns**, **Contract issues** (any `any`, `object`, unresolved imports), **ADR completeness**, **Recommendations**.
2. **Read the review report.** Revise if needed.
3. **Present to the user.** Show: path to `00b-PATTERNS.md`, the Mermaid module map inline, the contract file list, sub-agent review path and top findings.
4. **Ask:** "Reply `approved patterns` to proceed, or tell me what to change."
5. **On approval:** run `state-update.mjs --approve patterns`. Phase advances to `spec`.

Hook restriction during this phase: writes outside `.ai/spec-tdd/<slug>/` and `src/contracts/` are blocked.

## Phase 2 — Write SPEC and acceptance gates

State: `spec` (set by patterns approval). Create `01-SPEC.md` and `02-ACCEPTANCE.md` using templates. Requirements numbered (`R-001`), acceptance criteria numbered (`AC-001`), non-goals explicit.

Every requirement MUST have a `Source` column tracing to: source document (file + section), user request, architecture entry, or assumption. Requirements with no traceable source are likely hallucinated — flag for user review.

After writing SPEC, cross-check against BOTH `00-ARCHITECTURE.md` AND `00b-PATTERNS.md`: every contract, module boundary, ADR consequence, and constraint must appear as a requirement or explicit non-goal. Missing items indicate the SPEC is incomplete.

Ask questions only for truly blocking ambiguity. Otherwise make conservative assumptions and mark them in the SPEC.

Self-check the SPEC against `reference/spec-quality-rubric.md`.

When done, advance: `state-update.mjs --phase test_design`.

## Phase 3 — Design tests before implementation

State: `test_design`. Create `03-TEST_PLAN.md` and `04-TRACEABILITY.md` using templates and `reference/test-design.md`.

Requirements:

- Map every `R-*` and `AC-*` to at least one verification method: unit, integration, contract, E2E, static, build, manual, visual, emulator/simulator, or runtime smoke.
- Prefer integration/contract/E2E tests for user-visible behavior and public APIs (public APIs are defined in `00b-PATTERNS.md`).
- Use existing frameworks when present.
- Define the exact command that will fail before implementation and pass after.
- Include a **Red Test Protocol**: expected failing tests, expected failure reason, command, log path.

### Mandatory test categories

For every requirement, systematically derive tests across ALL of these. Mark each "covered" or "N/A with reason":

1. **Happy path** — basic success scenario.
2. **Boundary values** — zero/empty, min, min-1, max, max+1, single-element.
3. **Equivalence partitions** — divide input space, test one valid + one invalid per class.
4. **Error/failure paths** — at least 30% of tests are failure/negative. For every dependency: unavailable, error response, unexpected data, timeout.
5. **Error specificity** — every error assertion checks error type/class AND error code/status. Never bare `toThrow()`.
6. **Security inputs** — SQL injection, XSS, path traversal, command injection, SSRF, null bytes, control characters.
7. **Concurrency** — at least one test for shared mutable state.
8. **Resource cleanup** — connections, file handles, temp files, locks released on success AND failure.
9. **Side effects** — assert state changes beyond return values.
10. **Idempotency** — if applicable, call twice with same input, assert side effect happens once.

### Anti-pattern self-check

- No test that would pass even if function body were `throw "not implemented"`.
- No test with only `toBeDefined()` / `toBeTruthy()` / bare `toThrow()` assertions.
- No suite where >70% of tests are success cases.
- No test that asserts mock call counts as primary verification.
- Every error test checks error type AND code.

When done, advance: `state-update.mjs --phase red_tests`.

## Phase 4 — Write tests and prove red

State: `red_tests`. Write only tests, fixtures, mocks for external boundaries, and minimal test infrastructure. Do not implement feature logic.

Tests should target the public surfaces declared in `00b-PATTERNS.md`'s contract files. If a contract is wrong, fix `00b-PATTERNS.md` and re-request a patterns approval — do not silently change contracts.

Run the narrowest relevant command first, then broader verification. A red test is valid only if:

- it fails because the target behavior is missing or incorrect;
- the failure message points to the requirement/acceptance criterion;
- it is not failing because of syntax errors, broken imports, unavailable dependencies, wrong assumptions, flaky timing, or test harness mistakes;
- the assertion checks specific values/types, not just truthiness;
- if using mocks, the test would also fail with a stub returning the wrong value.

If a new test passes unexpectedly, treat as evidence the behavior already exists or the test is weak. Investigate and either update SPEC/traceability or strengthen the test.

Self-check against the **test suite completeness checklist** in `reference/test-design.md`. Every category must be "covered" or "N/A with reason".

When red tests are confirmed, advance: `state-update.mjs --phase tests_review`.

## Phase 4.5 — Tests review gate (human-required)

When `state.phase = tests_review` and `state.approvals.tests` is null:

1. **Invoke sub-agent.** Instruct it to read `01-SPEC.md`, `02-ACCEPTANCE.md`, `03-TEST_PLAN.md`, `04-TRACEABILITY.md`, and the test files. Verify:
   - Every `R-*` and `AC-*` has at least one mapped test.
   - The 10 mandatory test categories are covered (per `reference/test-design.md`).
   - No tautological tests (a test that would pass against a stub).
   - Error specificity: tests check error type AND code.
   - Failure case ratio (negative tests) ≥ 30%.
   - Boundary, security, concurrency, idempotency, cleanup, side-effect categories present.
   - Write `.ai/spec-tdd/<slug>/reviews/tests-review.md` with: **Coverage gaps (R-*, AC-* without tests)**, **Tautological tests detected**, **Missing categories**, **Recommendations**.
2. **Read and revise.** Add missing tests if the report flags gaps. Re-confirm red.
3. **Present to the user.** Show: test file list, traceability summary (X of Y requirements covered), sub-agent review path and top findings, the latest red test output.
4. **Ask:** "Reply `approved tests` to proceed to freeze, or tell me what to change."
5. **On approval:** run `state-update.mjs --approve tests`. Phase stays at `tests_review` (no auto-advance); next step is freeze.

Hook restriction during this phase: writes outside `.ai/spec-tdd/<slug>/` and test files are blocked.

## Phase 5 — Freeze tests and gates

After tests gate is approved, freeze. The freeze script will **refuse** if any of the three gate approvals is missing.

```bash
node "$HOME/.claude/skills/spec-tdd/scripts/freeze-tests.mjs" \
  --project . \
  --slug <feature-slug> \
  --verify "./scripts/verify.sh" \
  --files <test-file-1> <test-file-2> <e2e-or-contract-file>
```

Include all new/changed test, fixture, contract, snapshot, and verification files in `--files`. Also include `00b-PATTERNS.md` contract files under `src/contracts/` if you want them protected during implementation. Ensure `scripts/verify.sh` exists and runs the full final gate.

The freeze script writes `.ai/spec-tdd/state.json` with `phase: "implementation"` and preserves all three approvals. The frozen-files hook then blocks edits to frozen files; the Stop hook blocks stopping while verify fails.

## Phase 6 — Implement in loop until verification passes

State: `implementation`. Loop:

1. Read the latest red-test output and traceability row.
2. Make the smallest production change that could satisfy the behavior.
3. Run a focused test command.
4. If focused command passes, run `./scripts/verify.sh` or the verify command in state.
5. If anything fails, inspect the log and fix the root cause.
6. Refactor only while all tests stay green.
7. Update `06-IMPLEMENTATION_LOG.md` with decisions and commands.
8. Mark `05-TASKS.md` items done only after their mapped tests pass.

Do not stop voluntarily during `implementation` unless verify passes or you are genuinely blocked by missing external resources. The Stop hook will independently run verification and ask you to continue if it fails.

Production code must respect the public surfaces in `00b-PATTERNS.md` and the contracts in `src/contracts/`. If implementation reveals that a contract is wrong, stop and ask the user to approve a patterns revision — do not silently modify contracts.

## Phase 7 — Completion report

When verification passes:

- `state-update.mjs --phase done` (or the Stop hook does this automatically when verify exits 0);
- summarize what changed, tests added, commands run, and acceptance criteria satisfied;
- show remaining risks or manual checks, if any;
- include `git status --short` and mention any uncommitted files.

## Recommended `/goal` condition for long runs

For long implementations, tell the user they can run this before invoking the skill or immediately after red tests are frozen:

```text
/goal ./scripts/verify.sh exits 0, .ai/spec-tdd/state.json phase is done, frozen tests and acceptance gates are unchanged, no tests are skipped/weakened, and no TODO/stub/hardcoded test-only implementation remains; or stop after 20 turns with a clear blocked report.
```

The skill hooks provide deterministic local checks; `/goal` adds a session-level evaluator for longer loops.

---
> Source: [HagonChan/spec-tdd](https://github.com/HagonChan/spec-tdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
