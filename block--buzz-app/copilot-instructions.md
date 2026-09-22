## buzz-app

> Read [the contribution workflow](docs/contributing.md) for commands and validation.

# Contributor instructions for AI agents

Read [the contribution workflow](docs/contributing.md) for commands and validation.
For interactive product work, default to edit → human tries the running app →
adjust in the agreed worktree. Do not gate each feedback round on E2E, native
builds, or full validation. Use mandatory hooks and focused behavior checks not
covered by them; use existing CI for broad validation. `just iterate` is optional.
Run `just scan` only when explicitly requested or needed to reproduce a broad
integration failure, not as a routine pre-push or handoff gate.
Track deferred checks: **ready to try** is not **validated**. Check
auth/signing, persistence/migrations, protocol semantics, and destructive writes
before live use.

Files marked `FOUNDATION` require explicit human guidance before editing and
stricter review. Escalate needed changes rather than editing without authorization.

When reviewing CI or test-cost changes, inspect job-summary counts, elapsed wall
time, summed test execution time, and slowest-test/file evidence for regressions.

## Worktree creation

Before creating a worktree, run `git worktree list` and choose the existing
checkout whose local development configuration should be inherited. Immediately
after `git worktree add`, run this from the new worktree:

```sh
scripts/bootstrap-worktree.sh /absolute/path/to/source/checkout
```

Do not start development before bootstrap completes. The script copies the
git-ignored `.env.local` without overwriting an existing target, then uses that
worktree's Hermit proxy to run `bin/pnpm install --frozen-lockfile`. Do not copy
other ignored paths: Keychain credentials and pnpm's package cache are
machine-shared, while dependencies and build output are regenerated. Follow the
per-worktree hook setup in `docs/contributing.md` before committing or pushing.

## Engineering standard

Before editing, state the intended outcome and non-goals. Read the owning code,
callers, and relevant design docs; preserve documented product decisions and
ownership boundaries. Resolve answerable questions from evidence; ask before
deviating from agreed scope or product behavior.

Target **9/10+ for minimalness, elegance, and correctness**: the smallest complete
solution, clear ownership, and no known material defects. Prefer existing patterns
and subtraction. No opportunistic refactors, speculative abstractions, or new
features disguised as fixes. Before expanding into another shared subsystem or
adding alternate-adapter support, show the human the scope change and smallest
complete alternative. Require a current caller or explicit approval for adapter
parity. Review necessity separately from correctness; passing tests do not justify
scope growth. Split at real ownership boundaries, not by deleting safety coverage.

For non-trivial work, make that standard operational:

- Before coding, publish a short scope checkpoint: required behavior, non-goals,
  existing owners/platform support to reuse, expected files, and a rough production
  diff budget (separate from tests/docs). A small fix needs only a sentence, not a
  design ceremony.
- Use one implementation owner per end-to-end change. Reviewers challenge necessity
  as well as correctness; delegate bounded evidence/review, not competing rewrites.
  Review the first working slice before expanding the design, without blocking
  ordinary human UI feedback on a full validation cycle.
- Justify each new abstraction, lifecycle owner, timer, retry policy, or shared
  contract expansion against a current requirement. If the implementation materially
  exceeds the checkpoint, stop adding machinery and show the smallest alternative
  and any behavior tradeoff before continuing. Do not silently weaken agreed behavior.
- Assess the combined feature diff, including stacked PRs. Passing tests, splitting
  PRs, or already-invested work do not establish proportionality. Preserve required
  regression coverage; do not game the budget by deleting tests or compressing code.
  Keep speculative hardening and unrelated failures outside the task.
- Close with one verified end-to-end result and explicit remaining gaps, not a chain
  of green intermediate repairs presented as completion.

Keep files cohesive and group modules and tests by owner. Treat size as a review
signal, not a quota. Extract stable boundaries only when they simplify the
requested change.

Minimal does not mean happy-path-only. Handle relevant boundary inputs, failures,
recovery, and lifecycle transitions; consider concurrency, persistence, security,
and performance where the change affects them. Do not add machinery for
hypothetical requirements.

Validate the affected user contract, not just isolated helpers. Add regression
coverage for changed behavior and relevant failure paths; exercise real integration
boundaries where practical. Follow the contribution workflow's iteration and batch
gates, rather than adding full validation to every edit.

Self-review before handoff; seek independent review for risky changes before
integration. Report what changed, evidence tied to the checked snapshot, and
remaining risks or deferred checks. Green CI is evidence, not proof of user behavior.

Keep reviews convergent: consolidate actionable findings and clear exit criteria.
Block on concrete correctness, security, or agreed-contract defects; unrelated
hardening is follow-up. Reopen scope only when new evidence warrants it.

## Choose tests by behavior

Default to colocated Vitest tests. Mount React with React Testing Library for
component behavior; do not mock React hooks or implement a substitute lifecycle.
Use Node for logic/services and opt into jsdom only when a DOM is needed.

Before adding a browser journey, name the browser behavior or integration boundary
it proves that lower-layer tests cannot. Keep scenario matrices in the lowest
layer that preserves that contract; retain representative app wiring coverage.
Do not infer layout, native editing or cross-window correctness from a DOM emulator.
When moving coverage, map removed assertions to replacements and demonstrate that
the replacement catches the regression before deleting the browser case.

Make minimal fixture data the default and opt into larger datasets only for an
explicit scale, pagination or geometry contract. Share immutable builds and stateless
servers, never mutable test state, identities or browser contexts. Preserve large
datasets and isolated runners when scale or performance is the behavior under test.
Record browser cases added/removed, their browser-only justification, replacement
coverage and fail-then-pass evidence in the PR description. For test infrastructure
changes, report before/after setup and execution timings with the command, engine,
environment and checked snapshots; distinguish local measurements from hosted CI.
List deferred checks. Do not meet time budgets by skipping engines, dropping
failure paths or weakening assertions. Enforce these rules during agent review;
do not rely on contributors filling in a PR template. Follow the
[test-layer review rules](docs/contributing.md#choosing-a-test-layer).

## Deterministic tests

Tests must control the ordering they assert, not depend on runner speed.

- For intermediate states (loading, disabled, closing), hold the responsible
  operation with an explicit fixture gate or deferred promise. Observe that it
  started, assert the pending state, release it in `finally`, then verify recovery.
  A short artificial delay is not a synchronization primitive.
- Before capturing request counts, scroll anchors, or other baselines, establish
  the relevant lifecycle boundary: completed startup/catch-up, mounted result,
  applied layout, or observed event. Visible initial content does not prove that
  background work finished. Register event observers before triggering actions.
- Wait for observable conditions with retrying assertions, not fixed sleeps or
  immediate snapshots of asynchronous effects. Negative assertions need a
  completion barrier proving the work that could violate them has finished.
  Scope selectors to the semantic content being tested, not unrelated UI.
- When elapsed time is the behavior under test (expiry, debounce, retry), use a
  controlled clock and assert before/after the boundary. Keep real clocks for
  performance measurements; preserve their documented isolation and budgets.
- Do not hide failures with test retries, longer delays/timeouts, relaxed counts
  or tolerances, disabled animation, or broad error allowlists. Establish whether
  the defect belongs to the product, fixture, or assertion and repair that owner.
  Exercise adversarial ordering and the affected full test files in both browser
  engines for browser changes. Report exact validation scope and remaining gaps;
  a green run does not establish that all flakes are gone.

## Commit attribution and DCO

Every PR commit requires `Signed-off-by`. Use `git commit --signoff` with your
verified effective `git config user.name` / `user.email`; stop if missing or
incorrect. Preserve actual authorship; requesting or reviewing work does not
justify substituting the human's identity or adding their sign-off.

DCO, cryptographic signing, and co-author credit are separate; hooks do not supply
DCO. Audit **every commit against the PR base**, including after rebases or
cherry-picks. Preserve valid trailers; add only certifications you can make.
After repairs, verify the hosted **DCO Check** at the new head.

## Before pushing

- Use the agreed feature worktree and pinned `bin/` tools. Follow
  [hook setup](docs/contributing.md#pre-commit-checks) once per worktree;
  preserve custom hooks and never bypass failures.
- Refresh remote refs; confirm destination, base, and head. Review `git status`,
  the full PR diff, and `git diff --check` against the base. Include only intended
  files: no credentials, local configuration, or raw agent/session data.
- Follow the [validation workflow](docs/contributing.md#interactive-product-iteration).
  Documentation-only changes need content, link, and diff checks, not source
  builds. Keep incomplete work in draft with deferred checks listed; hook success
  or a draft push does not establish validation.
- Push the actual PR head ref. Never rewrite others' commits; authorized rewrites
  require `--force-with-lease`. Verify the remote head matches reported checks.
- Inspect current hosted checks and repository rules, including external checks.
  Resolve relevant failures before declaring readiness; obtain required reviewer
  and code-owner approval. This preflight is not automatic permission to merge.

Icons use Phosphor only, through `src/shared/design-system/icons`. Add individual exports as needed; icon and weight choices belong to the designer. The local lint and design checks enforce this import boundary.

---
> Source: [block/buzz-app](https://github.com/block/buzz-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
