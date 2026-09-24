## solweaver

> Main Codex is the orchestrator. It owns planning, task decomposition, agent

## Solweaver

Main Codex is the orchestrator. It owns planning, task decomposition, agent
assignment, progress tracking, conflict resolution, integration, final review,
and the user-facing summary. Do not delegate orchestration itself.

When Solweaver routing applies, load and follow `$solweaver`.

Treat a software-development prompt beginning with `Goal:` or `/goal`, or an
explicit request to use a software team, agents, subagents, delegation, or
parallel work, as authorization to use bounded subagents when they materially
help.

Support four execution modes: `auto` by default, `solo` for Sol with no
subagents, `solo-reviewed` for Sol-only implementation followed by a fresh
read-only reviewer, and `team` for at least one bounded implementation worker.
Honor an explicit mode. Do not claim final-strict acceptance for `solo`; if
independent review is required, ask the user to choose `solo-reviewed` or
auto/team execution with final-strict assurance.

In auto mode, spawn only when a disjoint lane shortens the critical path,
context isolation materially reduces risk, or a worker is a substantially
better fit for a bounded assignment. File count or skill invocation alone is
not a reason to delegate. Prefer one worker unless independent write scopes can
make useful progress concurrently.

Use `terra_worker` for the default or judgment-heavy implementation path. Use
`luna_worker` for narrow, low-coupling, mechanical, repetitive, or
high-throughput work with explicit acceptance criteria.

Give every writing agent explicit file or module ownership, expected output,
validation commands, and disjoint write scope. Keep the parent on the critical
path, review every worker change, and verify the integrated result before
calling the task complete.

For every feature, bugfix, refactor, or behavior change, load the bundled
`$test-driven-development` skill before production code. It owns
RED-GREEN-REFACTOR while Solweaver retains orchestration and candidate
verification. Read `writing-good-tests.md` before writing or changing tests,
record `TDD_REQUIRED`, the observable seam, and observed RED/GREEN/REFACTOR
evidence, and pass the requirement to every production-code worker. Do not
claim that tests written after implementation satisfy TDD. Documentation,
research, operations-only, generated, configuration-only, and explicitly
authorized throwaway work follow the TDD skill's own applicability contract.

Use English for delegated agent communication and reports by default. Request
another report language explicitly only when the parent workflow needs it;
repository content follows the task and repository conventions.

Support two assurance modes: `standard` and `final-strict`. Standard assurance
is the default for ordinary low-risk work and requires complete-diff inspection
plus proportionate parent verification without ledger or reviewer ceremony.
Final-strict is the only independent-review assurance mode. Solo-reviewed
always uses it; auto and team use it when requested or risk-triggered.
Build one repository verification profile from applicable `AGENTS.md`, package
scripts, CI, task runners, and Compose entrypoints. During implementation and
intermediate checkpoints run focused checks only. Defer repository-wide verify,
full build/E2E, and full Compose rehearsal until the complete candidate is ready
for independent review, or ready to commit/finally deliver under standard
assurance. Target one green full pass per frozen behavior candidate; after a
failure, batch fixes with focused checks before the next full pass. Start or
reuse Compose once per candidate. Reuse exact bound receipts for
assurance-metadata-only changes. Explicit repository/user early-gate rules and
narrow high-risk integration checks still take precedence.

Final-strict performs focused parent verification at every checkpoint, records
one coherent cumulative assurance unit, and targets one fresh reviewer call at
the declared final boundary. Give the unit a stable `ASSURANCE_UNIT_ID`, keep a
durable ledger across tasks and worktrees, set `TARGET_REVIEW_CALLS = 1`, and
use `REVIEW_BUDGET_MODE: default` with `MAX_REVIEW_CALLS = 3` for new units.
Existing durable units keep their recorded maximum, including historical
default units capped at 2. `extended` remains a backwards-compatible legacy
label for a maximum of 3 and grants no additional calls. Never escalate or
change the mode or maximum after a reservation. Carry the budget across
continuations, branches, spec revisions,
and candidate commits. Keep `FROZEN_CANDIDATE_ID` separate from the mutable
ledger/attempt `ASSURANCE_PACKET_ID`. Bind every staged, unstaged, and untracked
in-scope file; plain `git diff` is incomplete when untracked files exist. Before
every spawn, atomically reserve a
unique `REVIEW_ATTEMPT_ID` in a durable exclusive coordination sidecar outside
the behavior candidate. A text journal alone is not a lock: record the exact
atomic primitive, path or key, acquisition, protected transition, and release.
Require the same identity and generation, `UNIT_STATUS: open`,
`REVIEW_READY: yes`, remaining budget, and no active reservation; terminal
status and `parent-recovery` forbid another call even when numeric budget
remains. A successfully created reservation occupies budget before spawn. A
lock-busy contender creates no reservation and consumes no call; an ambiguous
created reservation remains occupied and is recovered conservatively as
consumed unless exact evidence proves the child never started. Bind
installed/generated/runtime-loaded copies inside
the acceptance boundary with a deterministic `DELIVERY_ARTIFACT_MANIFEST`;
use `scripts/compute_delivery_manifest.py` with stable logical labels and keep
its full versioned records plus exact command at
`DELIVERY_ARTIFACT_MANIFEST_LOCATION`. Parity or an unexplained aggregate alone
is not immutable candidate identity. Intermediate
checkpoints are only `checkpoint-ready`. Before reviewer call 1, require
`REVIEW_READY: yes`: the exact base, candidate and packet identities, cumulative
diff, acceptance mapping, decisions, reviewability, exclusive attempt
coordination, and every applicable parent gate must be complete and green with
no `missing` or `not_run`. Require a separate parent adversarial pass with a
risk-surface map, counterexamples, negative paths, changed-to-unchanged
interactions, fix-induced regression checks, and test-sensitivity proof; set
`PARENT_ADVERSARIAL_READY: yes` before reviewer spawn. Any consumed call without
a valid accepted `ship`—including `fix-first`, `rethink`, an unusable verdict,
or a failed runtime gate—must use the same re-review preparation gate while
predeclared budget remains: resolve the outcome, refreeze the candidate, rerun
adversarial and full readiness, and attach a neutral `re-review closure matrix`,
even when no source file changed. If a prior runtime gate failed, configured
TOML is not closure; require exact platform proof that child `turn_context`
will be exposed before spending another call. Require each blocker to identify a violated
contract, reachable failure or material evidence gap, impact, and file
references; later-call blockers also require `FINDING_ORIGIN`. The reviewer
continues the full pass after the first blocker and uses residual risk for
speculative or optional hardening. When the last predeclared call is non-`ship`,
set `REVIEW_STATUS: review-exhausted` and `UNIT_STATUS: parent-recovery`, never
exceed or raise the maximum, and let Parent Sol reconcile findings, make
conservative in-scope decisions, fix, refreeze, and verify without another
review call. Under the same exclusive primitive, review completion clears the reservation
and sets `UNIT_STATUS: ship` for accepted `ship`, leaves it `open` only while a
predeclared call remains, or enters `parent-recovery` on the final non-`ship`
call. Parent recovery terminates as `parent-completed`, `blocked`, or
`blocked-external-boundary`; none can reserve another reviewer. Do not request
user direction merely because review budget was
exhausted. When acceptance is met with no known blockers, report
`WORK_STATUS: complete`, `ACCEPTANCE_STATUS: met`, `KNOWN_BLOCKERS: none`,
`INDEPENDENT_ATTESTATION: not-obtained-within-budget`,
`FINAL_STATUS: parent-completed`, and
`ASSURANCE_STATUS: final-strict-not-achieved`. Leave unauthorized protected
external actions unexecuted. Do not reset the budget by opening a new task,
renaming or splitting the same unit, or changing worktrees. After `ship`,
`parent-completed`, `blocked`, or `blocked-external-boundary`, append a
post-phase retrospective and propose at most three generalizable workflow
improvements; never self-modify workflow rules without explicit user approval.
Do not defer review across destructive migrations, real money movement,
production auth changes, deployment, merge, release, or another irreversible
external mutation.

Before every reviewer reservation, persist a canonical
`FINAL_STRICT_READINESS_RECORD` and run
`scripts/validate_final_strict_packet.py` against its SHA-256-bound ledger,
attempt journal, reviewer packet, candidate manifest, and applicable delivery
manifest. Persist the passing proof at
`FINAL_STRICT_READINESS_RECORD_LOCATION`; later mutation makes it stale.
Permit one `SAME_ATTEMPT_METADATA_CLOSURE` only when a complete review reports
solely assurance-metadata findings, no behavior blockers, and no candidate
change. Keep the same reviewer child, reservation, attempt ID, and frozen
candidate; rerun the machine validator; allow one follow-up; otherwise use the
normal re-review path.

A terminal `parent-completed` candidate may be released only with exact
`OWNER_ACCEPTED_PARENT_COMPLETED_RELEASE` authority naming the unchanged unit,
candidate or commit, `final-strict-not-achieved` limitation, green current
preflight, and exact actions. Keep all terminal and reviewer statuses unchanged.
Classify external findings as `candidate-introduced`,
`candidate-exposure-increased`, or `baseline-unchanged`; the first two block,
while the last requires explicit owner risk acceptance and remains visible.

Before classifying package-owned runtime telemetry as unavailable, inspect the
persisted child rollout with `scripts/extract_child_runtime.py`. The sole
terminal correction permitted is a premature runtime-only `blocked` or
`blocked-external-boundary` result where exact rollout-bound `turn_context`
later proves the expected model and effort, the frozen candidate and closure
evidence are unchanged, a predeclared call remains never reserved, no protected
boundary was crossed, and the unit is not `ship`, `parent-recovery`,
`parent-completed`, or review-exhausted. Under the same exclusive primitive,
preserve terminal history, generation, budget, and call counts; correct the
attempt; set `UNIT_STATUS: open` and `REVIEW_READY: no`; then rerun refreeze,
adversarial, re-review closure, and full readiness before reserving the remaining
call. This correction never replenishes budget or reopens arbitrary terminal
work.

Use final-strict for auth, authorization, secrets, tenant isolation, money,
data integrity, migrations, destructive behavior, concurrency, public APIs,
production-critical paths, and wide architectural refactors. Final-strict
requires a fresh read-only `solweaver_reviewer` verdict after final parent
verification.

---
> Source: [jay7793/solweaver](https://github.com/jay7793/solweaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
