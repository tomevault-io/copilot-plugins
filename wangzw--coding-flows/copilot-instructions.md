## coding-flows

> |


# coding-flows — Coder / Reviewer cycle

Paired-agent GitHub workflow for a single repo. **One invocation = one cycle**
for one role. Long-running monitoring is handled by the wrapping scheduler
(`slock reminder` or `/loop`), not by this skill staying alive.

## Quick start

```
role=coder       # turn issues into PRs and merge approved PRs
role=reviewer    # review open PRs until they meet merge bar
```

The caller MUST specify the role. If not specified, **stop and ask**.

## Per-repo configuration

The skill reads `.coding-flows.json` from the repo root. It controls: acting user,
merge method, scope rules (which standards apply to which paths), high-risk
label rules, AC-mapping / LGTM-coverage requirements, and flaky-test policy.
If the file is absent, sensible defaults apply (LGTM coverage and AC mapping
stay required — those are core invariants).

- Schema: **[references/config-schema.md](references/config-schema.md)**
- Example: **[examples/coding-flows.example.json](examples/coding-flows.example.json)**

## Pre-flight (every cycle)

1. `gh auth status` — must match `acting_user` from `.coding-flows.json` (or current
   authenticated user if no config). On mismatch, abort with `auth-broken`.
2. cwd is inside a GitHub repo: `git rev-parse --show-toplevel` + `gh repo view`.
   Worktree cwds are valid — `find_repo_config` falls back to the main repo via
   `git rev-parse --git-common-dir`.
3. **Main-checkout `git status` is no longer pre-flight-blocking.** Per-issue
   work happens in worktrees; the main checkout's state is independent.
   Per-worktree state is checked per-item in Coder Phase B / E.
4. Coder only: `scripts/coding-flows-worktree-prune` — retire worktrees whose PRs
   have merged or closed since the last cycle (best-effort).
5. Read project conventions: `CLAUDE.md` / `AGENTS.md`, `commitlint.config.*`,
   `.github/PULL_REQUEST_TEMPLATE.md`, `.github/CODEOWNERS`, build commands
   from `Makefile` / `package.json` / `pyproject.toml`.
6. **Mandatory scope detection.** Run `scripts/coding-flows-detect-scope` against
   the diff (Coder: working branch; Reviewer: target PR). It enumerates which
   coding-standards skills + invariant docs apply per `.coding-flows.json`
   `scope_rules`. **Read each one into context before Phase B/C.** This closes
   the "invariant never loaded" failure mode.

## Cycles

- **Coder** — see **[references/coder-cycle.md](references/coder-cycle.md)**.
  Per-issue **worktree** isolation — worktrees default to siblings of the
  main checkout at `<parent-of-main-repo>/<branch-slug>/`; see
  [references/worktrees.md](references/worktrees.md). Within one cycle the
  Coder is serial (one build/test at a time) but multi-task across items,
  with **strict two-stage ordering**: process every actionable PR
  (address review comments → fix CI failures → merge) before touching any
  issue. When an item hands off to CI or Reviewer, the cycle rotates to
  the next actionable item rather than blocking. **The scheduler's
  interval is polling frequency, not a per-cycle time budget**: take
  however long an item needs (20 min, an hour) before rotating —
  partial work and draft-PR shortcuts are not part of the workflow.
  Iteration frequency comes from the wrapping scheduler (`slock
  reminder` / `/loop`), not from this skill.
- **Reviewer** — see **[references/reviewer-cycle.md](references/reviewer-cycle.md)**.
  Default-deny adversarial stance. Builds a **local** review plan (under
  `~/.cache/coding-flows/<owner>/<repo>/pr-<N>/`); the plan never touches the PR.
  Publishes only `/lgtm` with a coverage marker (compact, machine-verifiable)
  or `--request-changes` with natural-language findings.

## Merge gates (hard, machine-checked)

A PR is merge-ready only when **all** pass. Enforced by `scripts/coding-flows-merge` —
agents **must not** call `gh pr merge` directly.

1. **CI green** on the latest commit (flaky-test policy applied first — see
   [references/flaky-test-policy.md](references/flaky-test-policy.md)).
2. **Linked issue** present in PR body (`Closes #N` / `Fixes #N` / `Resolves #N`).
3. **AC mapping table** present and complete — see
   [references/ac-mapping.md](references/ac-mapping.md).
4. **LGTM coverage** — the union of `acs=`, `invariants=`, and
   `risks-reviewed=` claims across current-head `/lgtm` markers covers every
   AC ID, every triggered invariant, and every Coder-declared risk category.
   See [references/review-plan.md](references/review-plan.md).
5. **Current LGTM** — most recent `/lgtm` comment is bound to the current
   head SHA; any new push invalidates the prior LGTM.
6. **No unresolved review threads.**
7. **Squash or rebase only** (per `merge.method` in config; allowance against
   repo settings is verified live or via `CODING_FLOWS_REPO_ALLOWS`).
8. **High-risk PRs require dual LGTM** from two distinct reviewer markers —
   see [references/high-risk-pr.md](references/high-risk-pr.md).
9. **Risks declared** — `## Risks` table present with structured
   Category/Description/Mitigation rows + matching HTML marker.
10. **Scope envelope** — every changed file is claimed by an AC mapping row,
    a `## Support files` entry, or `scope_envelope.exclude_paths`.
11. **Test plan complete** — every bullet in the PR body's `## Test plan`
    section is explicitly `- [x]`. Plain bullets (no checkbox), `- [ ]`,
    `- [.]`, or any other non-`[x]` form blocks merge. Coder must either
    run the manual verification and mark `- [x]`, or remove the entire
    bullet line.
12. **Issue↔PR AC count parity** — PR body's `## AC mapping` table has
    at least as many rows as the linked issue's `## Acceptance criteria`
    checklist. Catches Coder silently dropping AC items between issue
    and PR. Skipped (with warning) if the linked issue body can't be
    fetched.

Full reference: **[references/merge-gates.md](references/merge-gates.md)**.

## Adversarial Reviewer

Reviewer cycles run with a **default-deny** stance: the goal is to find reasons
NOT to approve. `/lgtm` is signed only when every line of the review plan has
documented evidence. Prompt template: see
[references/reviewer-cycle.md](references/reviewer-cycle.md) §"Adversarial prompt".

For high-risk PRs, a **second reviewer agent** is dispatched with the same
adversarial prompt but blind to the first review's findings — both must
independently sign `/lgtm` before merge.

## Self-approval / LGTM signal protocol

When Coder and Reviewer authenticate as the same GitHub user, formal
`--approve` is blocked. Approval is signaled by a top-level PR comment whose
**first line is exactly `/lgtm`** with a marker binding it to head SHA and
enumerating coverage:

```
/lgtm

<!-- coding-flows:lgtm sha=<full-head-sha> reviewer=<reviewer-id> acs=AC-1,AC-2 invariants=name1,name2 risks-reviewed=migration,perf -->
<1-3 sentence summary of evidence — what AC items, what invariants, what
risks, what edge cases were verified.>
```

Required fields: `sha`, `reviewer`, `acs`, `invariants`. `risks-reviewed=`
is parsed when present (covers Gate 4's risk-coverage check; older markers
without it are still parsed for backwards compat — coverage simply has
nothing to enforce on that axis).
The visible body stays short — comment format rules in
[references/pr-comment-format.md](references/pr-comment-format.md).

`scripts/coding-flows-check-lgtm` parses the marker; `coding-flows-verify-lgtm-coverage`
confirms `acs=` covers the AC mapping table and `invariants=` covers every
triggered invariant.

## PR comment format discipline

Every comment the skill posts must follow
[references/pr-comment-format.md](references/pr-comment-format.md):

- ≤ 25 visible lines, machine data in HTML comments only
- One `/lgtm` per reviewer per head SHA
- No status-update / "plan v=N" / "verdicts filled" comments
- Blocking findings go in `gh pr review --request-changes` body (prose +
  file:line bullets), not in a separate top-level comment

The Reviewer's full plan is a local file under
`~/.cache/coding-flows/<owner>/<repo>/pr-<N>/`; **never** paste it into the PR.

## Notification protocol (`needs-human`)

When the cycle needs human input, it adds `needs-human` label + posts a marker
comment. See **[references/notification-protocol.md](references/notification-protocol.md)**
for reason tags (`ambiguous-issue`, `disagreement`, `iteration-cap`,
`repeated-ci-failure`, `dual-review-disagreement`, etc.) and dedupe rules.

## Hard guardrails

- **Never** run `gh pr review --approve` on a PR authored by the same user.
- **Never** merge except through `scripts/coding-flows-merge` — it enforces all 11 gates.
- **Never** force-push to a branch with open Reviewer comments not yet addressed.
  Use additional commits.
- **Never** close an issue you can't actually fix; comment instead.
- **Never** edit a test to make it pass. If a test is wrong, file an issue.
- **Never** commit secrets, `.env` files, or credentials.
- **Never** bundle unrelated changes into a PR — file a follow-up issue and a
  separate PR.
- **Never** add a `Co-Authored-By:` trailer to any commit — neither in direct
  commits nor in commits made by sub-agents you dispatch.
- If `gh auth status` reports unauthenticated → abort the cycle.
- If cwd is not a GitHub repo → abort the cycle.
- If `git status` shows unrelated dirty state → abort the cycle.

## Scripts

All under `scripts/`. Each has a pure verification lib under `scripts/lib/`
that takes JSON via stdin or arg (so tests can run with fixtures).

| Script | Purpose |
|--------|---------|
| `coding-flows-detect-scope`          | List skills/standards triggered by a diff |
| `coding-flows-label-risk`            | Apply `high-risk:*` labels per config rules |
| `coding-flows-verify-ac-mapping`     | Parse PR body, validate AC mapping table |
| `coding-flows-verify-risks`          | Parse PR body, validate `## Risks` section (Gate 9) |
| `coding-flows-verify-scope-envelope` | Confirm every changed file is claimed (Gate 10) |
| `coding-flows-verify-issue-ac`       | Gate 12: PR body AC mapping covers every issue AC item |
| `coding-flows-coder-plan`            | Manage Coder's local working plan: init/path/render/validate |
| `coding-flows-clarification-status`  | Determine if a `clarify`-state issue is now actionable |
| `coding-flows-check-lgtm`            | Parse `/lgtm` markers; confirm head-binding |
| `coding-flows-verify-lgtm-coverage`  | Demand `acs=`/`invariants=`/`risks-reviewed=` claims cover Coder's declarations |
| `coding-flows-classify-pr`           | Coder Phase A: canonical PR state (`address-changes` / `ready-to-merge` / `wait-*`) — use this instead of inspecting `reviewDecision` |
| `coding-flows-classify-pr-reviewer`  | Reviewer Phase B: maps full merge --dry-run state to `needs-review` / `needs-resign` / `needs-second-lgtm` / `idle-*` |
| `coding-flows-verify-test-plan`      | Gate 11: every `- [ ]` in `## Test plan` is checked or removed |
| `coding-flows-show-coverage`         | Emit exact `acs=`/`invariants=`/`risks-reviewed=` values for the Reviewer's LGTM marker |
| `coding-flows-revoke-lgtm`           | Reviewer: post a top-level comment with revoke marker (works on self-PRs; supersedes prior `/lgtm` via `check-lgtm`) |
| `coding-flows-pr-create`             | Wrap `gh pr create`; guarantees `--assignee @me` lands. Use this instead of raw `gh pr create`. |
| `coding-flows-issue-create`          | Wrap `gh issue create`; guarantees `--assignee @me` lands. Use this instead of raw `gh issue create`. |
| `coding-flows-fetch-for-reviewer`    | Emit a comment-filtered PR view for dual-reviewer blindness |
| `coding-flows-worktree-create`       | Open a managed worktree for an issue |
| `coding-flows-worktree-list`         | List all managed worktrees + branches + heads |
| `coding-flows-worktree-path`         | Resolve a branch's managed worktree path |
| `coding-flows-worktree-remove`       | Tear down a managed worktree (auto-called by `coding-flows-merge`) |
| `coding-flows-worktree-prune`        | Cycle pre-flight cleanup of MERGED/CLOSED worktrees |
| `coding-flows-worktree-sync`         | Rebase the current worktree onto its base ref |
| `coding-flows-merge`                 | Compose all gates → `gh pr merge` (hard blocker); cleans up worktree on success |

Reference: **[references/scripts.md](references/scripts.md)**. Tests:
`tests/run-all.sh`.

## Output protocol — end of every cycle

Print a one-screen summary the wrapping scheduler can read at a glance:

```
coding-flows cycle  role=<coder|reviewer>  repo=<owner/name>  ts=<ISO8601>

Open work: PRs=<count>, issues=<count>   ← Coder cycle: both Phase A queries
                                            (gh pr list / gh issue list);
                                            Reviewer cycle: PRs only.

Processed:
  - issue #N: <one-line action>
  - PR    #M: <one-line action>

Skipped:
  - PR    #K: idle-merge-ready   (Coder owns next step)
  - PR    #L: wait-ci             (Reviewer will pick up automatically once green)
  - PR    #M: wait-review         (waiting for Reviewer cycle)
  - issue #J: wait-author         (clarification question already labelled `needs-human`)

Needs human:
  - issue #J  reason=ambiguous-issue       notified=label+marker
  - PR    #L  reason=repeated-ci-failure   notified=already-labeled
```

### Full coverage is mandatory

For a **Coder cycle**, every PR returned by `gh pr list --assignee <me>
--state open` AND every issue returned by `gh issue list --assignee
<me> --state open` MUST appear in exactly one of `Processed:`,
`Skipped:`, or `Needs human:`. The `Open work:` counts let a reader
verify this at a glance: if `issues=6` and the body has zero `issue
#…` rows, the cycle was buggy (Stage 2 was skipped, or its results
were dropped from the summary). Observed failure mode: the Coder
processed only PRs for many consecutive cycles while issues #266,
#268, #276–#279 went untouched — because the output had no slot to
make the omission visible.

For a **Reviewer cycle**, the same rule applies to PRs only — issues
are out of scope. `Open work:` therefore omits the `issues=` segment.

### Creating items: use the wrappers (assignee enforced)

Both Phase A queries filter by `--assignee`. An un-assigned PR or
issue is INVISIBLE to the next cycle. The Reviewer never sees
un-assigned PRs (review never starts); the Coder never sees
un-assigned issues (full-coverage rule above can't report them).

To make the correct behavior the path of least resistance, use the
wrappers instead of the raw gh commands:

- **`coding-flows-pr-create ...`** instead of `gh pr create ...`
- **`coding-flows-issue-create ...`** instead of `gh issue create ...`

Each wrapper passes every argument through to gh, injects
`--assignee @me`, then verifies the assignee landed (with a fallback
`gh pr edit --add-assignee @me` if not). Drop-in compatible.

If for some reason you must call raw `gh pr create` / `gh issue create`,
you are responsible for including `--assignee @me` AND running the
same verification + fallback. Doc-only fixes for this rule regressed
four times in one session — that's why the wrappers exist:

- PRs #291/#293 cluster
- PRs #417/#419/#421/#423 + issues #418/#420/#422 cluster
- PR #431 + issue #430
- PR #433 + issue #432

Detailed Phase D template: [references/coder-cycle.md](references/coder-cycle.md).

### Skipped reasons are scoped to item type — do not mix

PR-side `Skipped:` reasons MUST be one of the classifier enum values.
For Coder cycles, use `coding-flows-classify-pr`; for Reviewer cycles,
use `coding-flows-classify-pr-reviewer`. The classifier output is the
ONLY source of state for PR rows — do not classify by head-SHA delta,
"nothing changed since last cycle", or any other heuristic. Run the
classifier for **every** PR in the input list, every cycle.

| Item type | Allowed `Skipped:` reasons | Source |
|-----------|---------------------------|--------|
| **PR** (Coder cycle) | `wait-ci`, `wait-review`, `wait-lgtm-fresh`, `merge-conflict`, `wip` | `coding-flows-classify-pr` |
| **PR** (Reviewer cycle) | `idle-merge-ready`, `idle-coder-blocked` | `coding-flows-classify-pr-reviewer` |
| **issue** (Coder cycle) | `wait-author`, `blocked-by-dep`, `out-of-scope`, `next-cycle` | local judgment |

Issue-side `Skipped:` reasons — exact definitions:

- `wait-author` — **clarification asked** of a human, `needs-human`
  label is on the issue, no new author response yet. The Coder
  re-checks each cycle for a new author comment. This reason is
  ONLY for genuine clarification needs — not for "work is done,
  waiting for someone to close it." See the "Coder closes its own
  completed issues" rule below.
- `blocked-by-dep` — issue concretely depends on another issue / PR
  that has not yet merged or been resolved. Name the dependency in
  the row (e.g. `blocked-by-dep #279 (allow-list pruning)`). NOT for
  "I'd rather finish unrelated work first" — see anti-patterns below.
- `out-of-scope` — issue is not for the Coder to do at all (e.g. spec
  decision pending, requires human research, owned by another team).
  Should be rare; once marked, expect it to stay marked across many
  cycles. NOT for "deferring to next cycle".
- `next-cycle` — valid Stage 2 issue, Coder is serial (one Stage 2
  issue picked per cycle), this one wasn't chosen. The next cycle
  re-evaluates from Phase A; no human action implied. This is the
  correct reason for "issue I would have worked on but had to pick
  another one first." Optional `(reason)` suffix to note why a peer
  issue was prioritized (e.g. `next-cycle (smaller scope picked first)`).

A Reviewer cycle emitting `wait-author` for a PR is a bug: the
Reviewer must call `coding-flows-classify-pr-reviewer` and use its
enum.

### Coder closes its own completed issues

When every AC of an issue is covered by merged PRs and there is no
further Coder work, the Coder **closes the issue directly**:

```
gh issue close <N> --comment "Closed: all named ACs covered per the
coverage summary above. <one-line cross-link to the merging PRs>."
```

The Coder does NOT skip the issue with `wait-author` "awaiting human
close." Humans are only involved when a real question needs human
judgment (spec decision, research, ambiguity). A done issue is the
Coder's responsibility to close.

Auto-close via a PR's `Closes #N` body line still handles the common
case: one PR fully closes one issue. The `gh issue close` step is for
umbrella issues that span multiple PRs — when all named sub-ACs land,
the umbrella stays open unless the Coder closes it.

Observed anti-pattern: issues #288, #289, #292, #295 had every named
AC closed by merged PRs and a Coder-posted coverage summary, but were
left OPEN with `wait-author` in cycle outputs. The four were closed
manually after they sat for hours. Don't repeat this.

**Anti-pattern — do not invent reasons to skip Stage 2.** Observed in
a transcript: Coder finished PR work (PR now in `wait-ci`), then
marked all 6 open issues `out-of-scope` with reason "PR-side backlog
has priority while #258 still needs to land". This is incorrect: once
every Stage 1 item is in a waiting state, Stage 2 MUST engage — that
is the entire premise of the strict two-stage ordering. "PR is in CI"
is the *trigger* for Stage 2, not a reason to defer it. If you find
yourself writing temporary-sounding reasons after `out-of-scope` /
`blocked-by-dep`, you are skipping Stage 2 with a fig leaf — pick the
top-priority `start` issue and run Phase A1 → D instead.

**"Needs human" has a precise meaning** — only items where the cycle
triggered the [notification protocol](references/notification-protocol.md):
the `needs-human` label was applied to the issue/PR and a marker comment
was posted. The human filters their GitHub by `label:needs-human` and
acts via that channel.

It is NOT a catch-all for "I can't act on this; someone else has to".
Things that look like that but belong in `Skipped`, not `Needs human`:

- "Reviewer cycle needed on PR #N once CI clears" → **Skipped: wait-ci**.
  The Reviewer cycle will pick it up automatically when CI is green;
  no human is needed.
- "PR has stale LGTM, awaiting reviewer to re-sign" → **Skipped:
  wait-lgtm-fresh** (Coder cycle) or **needs-review** (Reviewer cycle —
  this is your queue, not a skip).
- "Issue awaiting clarification response from author" → **Skipped:
  wait-author** (issue row only). The `needs-human` label is already on
  the issue from when the question was asked; don't re-report each cycle.
- "PR is in `wait-review`" → **Skipped: wait-review** (Coder cycle).
  Reviewer cycle owns next step.

Use `Needs human` only for fresh notifications this cycle triggered
(reason tags: `ambiguous-issue`, `disagreement`,
`dual-review-disagreement`, `iteration-cap`, `repeated-ci-failure`,
`broken-main`, `merge-blocked`, `auth-broken`, `unrelated-dirty-tree`,
`ac-mapping-unresolvable`, `invariant-violation-unfixable`,
`scope-too-large`, `scope-mismatch`).

## Wrapping for continuous monitoring

This skill runs **one cycle** per invocation. Wrap with:

- `slock reminder schedule --recur "every 10 minutes" --message "run coding-flows with role=coder"`
- `/loop` for tighter local polling within a single Claude session.

---
> Source: [wangzw/coding-flows](https://github.com/wangzw/coding-flows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
