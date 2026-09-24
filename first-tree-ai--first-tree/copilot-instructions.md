## first-tree

> `@first-tree/qa` contains the First Tree-specific assets used by the shipped `first-tree-qa` skill: tier rules, reusable

# QA Package Instructions

`@first-tree/qa` contains the First Tree-specific assets used by the shipped `first-tree-qa` skill: tier rules, reusable
briefings, natural-language QA cases, environment recipes, evidence guidance, templates, and fixtures. The skill owns
the core QA principles and lifecycle; this package is the repository-specific authority for how First Tree applies them.

It is not a test runner, CLI, or CI gate. Stable deterministic behavior belongs in product tests, recurring agent
behavior belongs in `@first-tree/skill-evals`, and live or judgment-dependent behavior may use focused or full QA.

## Tier Selection

Choose the lowest tier and narrowest affected scope that can support the requested conclusion, and record both before
setup. Use changed paths, public-contract impact, statefulness, cross-surface reach, and failure blast radius; line count
is only a supporting signal.

| Tier | Use it for | Environment | Maximum honest conclusion |
| --- | --- | --- | --- |
| `test-only` | Localized deterministic integration/regression checks or an explicit test request | Run matching package/named tests; use `pnpm test` only for repository-wide or shared-input scope | Only the reported automated checks passed or failed |
| `focused-local` | Ordinary feature validation, defect reproduction, or focused performance work | Reuse the QA warm environment and start only affected surfaces plus necessary adjacent boundaries | Only the observed paths passed or failed under the recorded local conditions |
| `full-isolated` | Release-sensitive, clearly major/high-risk, cross-surface, or explicitly isolated QA | Take an exclusive clean task slot in the QA warm environment for the selected scope | Only the completed isolated scope passed or failed; release-wide only when requested |

Do not select `full-isolated` merely because Docker is available, a committed case exists, or the request is vague.
`full-isolated` strengthens isolation; it does not automatically broaden validation to every product surface. A lower
tier may recommend escalation, but it must not claim release readiness or silently weaken a case that requires the full
tier.

## Environment Reuse

- Keep one QA-owned warm environment outside the source repository. Retain compatible infrastructure, caches, images,
  and dependencies after a run instead of rebuilding or tearing them down by default.
- Reuse the same task slot across retries and new target revisions. Before each reuse, record the warm-environment ID,
  task key, exact target, health, profile compatibility, and mutable-state baseline.
- Between tasks, release the task lease and reset task-owned data, namespaces, ports, processes, and credentials; do not
  destroy the warm environment. A retained environment is a reported residual state, not failed cleanup.
- Never reuse operator/customer data, logged-in browser profiles, writable provider homes, or unattributable mutable
  state. If reset or attribution is not credible, repair the slot, narrow the claim, or report `BLOCKED`/`INCONCLUSIVE`.

## Invariants For Every Tier

A result cannot be `PASS` when an applicable invariant is violated:

1. Resolve and report the exact target, selected tier, scope, commands or product paths, and evidence boundary.
2. Do not modify product source while testing. Product fixes and committed case maintenance are separate tasks.
3. Require evidence from the boundary actually claimed: test output for `test-only`; real relevant product behavior for
   `focused-local`; scoped isolated and real product evidence for `full-isolated`.
4. Keep retained run artifacts outside the source repository and redact secrets, credentials, private sessions, and
   user data.
5. Separate target failures from environment, dependency, credential, provider, platform, data-precondition, or
   insufficient-evidence failures.
6. Report exactly one status and one case disposition, and state task reset plus retained warm-environment state honestly.

## Tier Requirements

### `test-only`

- Run the matching package or named suite for a localized deterministic change. Run `pnpm test` when the requested
  conclusion or changed shared inputs require repository-wide validation.
- Record the exact command, exit code, duration when available, and failing test identifiers or output.
- Do not create a Docker cell, capability matrix, QA plan, or `QA READY` claim.
- A passing test command is not live-product, provider, cross-surface, performance, or release evidence.

### `focused-local`

- Prefer an exact-target worktree in the existing task slot. Reuse compatible task services across retries and start only
  affected surfaces plus the nearest boundary required by the validation question.
- Local non-isolated startup is allowed. Inventory the warm environment and any reused dependency or service first,
  record its owner/config/state/health, and do not mutate valuable or operator-owned data. Use task-owned data/config for
  write paths.
- Establish Build, Run, Drive, Observe, Measure, and Reset for the in-scope surfaces before executing the planned
  behavior. A lightweight `plan.md` may then record the focused scope.
- Docker is optional. Never reuse an operator's logged-in browser/provider session or mount writable host credential
  homes solely for convenience.
- Stop and report limitations if shared state makes attribution, cleanup, or a `PASS` conclusion unreliable.

### `full-isolated`

- Use an exclusive task slot in the QA-owned Docker-backed warm environment and an exact-target worktree, not the
  operator's checkout or shared mutable services. Explicit native, device, or provider bridges are allowed only when an
  in-scope surface cannot live credibly inside Docker.
- Select affected surfaces and critical adjacent boundaries from the PR/requirement before setup. Build, run, drive,
  observe, measure, and reset only that isolated scope; initialize every product surface only for an explicit
  release-wide request.
- Write the formal task QA plan after the selected isolated scope reaches `QA READY`. A provisional scope and reuse
  record may exist before readiness.
- A blocked in-scope capability does not stop safe independent probes within that scope. Preserve the scoped matrix,
  evidence, and available performance samples before reporting, then reset task-owned state while retaining the warm
  infrastructure.

## Lifecycle

1. Resolve the exact target and enough repository, issue/PR/design, Context Tree, release, CI, and QA context to choose a
   tier and affected scope.
2. Record the tier, scope, and rationale, then inspect and reuse the warm environment before preparing only the missing
   capabilities and evidence path.
3. Record scope: a command list for `test-only`, a post-capability focused plan for `focused-local`, or a post-scoped
   `QA READY` formal plan for `full-isolated`.
4. Execute through the selected boundary, adapt to live facts, and retain evidence tied to conclusions.
5. Report status, tier, scope, performance proportional to the tier, limitations, artifact paths, task reset, retained
   environment state, and case disposition.

Use one disposition: `no-change`, `candidate-new-case`, `candidate-case-update`, `move-to-product-test`,
`move-to-skill-eval`, or `merge-or-retire`. Apply the recommended change only in a separate maintenance or product-work
task.

## Result Statuses

- `PASS`: every check in the reported tier and scope completed with sufficient evidence and no attributable product
  issue was found.
- `FAIL`: a reproducible defect or deterministic test failure attributable to the exact target was found. Produce a bug
  artifact when reproduction detail is needed.
- `BLOCKED`: environment or external preconditions prevented required setup or validation.
- `INCONCLUSIVE`: evidence is incomplete, unstable, interrupted, contradictory, or unattributable.

## Package Boundaries

- Put First Tree case authoring guidance and reusable cases under `cases/`.
- Put tier-specific repository guidance under `briefings/`.
- Put local/full environment and provider-bridge recipes under `environment/`.
- Put evidence, performance-observation, and redaction guidance under `observability/`.
- Put minimal run artifact templates under `templates/`.
- Put reusable, non-run-specific assets under `fixtures/`.
- Do not add a public runner, lifecycle CLI, or CI gate without a separate design decision.

---
> Source: [first-tree-ai/first-tree](https://github.com/first-tree-ai/first-tree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
