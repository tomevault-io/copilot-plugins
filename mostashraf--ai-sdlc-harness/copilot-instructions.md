## ai-sdlc-harness

> This project is a Claude Code harness that orchestrates multi-agent development workflows for User Stories / Issues across multiple repos. Supports multiple work item providers (Azure DevOps, Jira, GitLab, GitHub, Zoho, local Markdown) and git providers (Azure DevOps, GitLab, GitHub, gh-cli, glab-cli) via a provider adapter layer. No application code lives here — only agents, skills, hooks, and context.

# AI-Driven Development Workflow

This project is a Claude Code harness that orchestrates multi-agent development workflows for User Stories / Issues across multiple repos. Supports multiple work item providers (Azure DevOps, Jira, GitLab, GitHub, Zoho, local Markdown) and git providers (Azure DevOps, GitLab, GitHub, gh-cli, glab-cli) via a provider adapter layer. No application code lives here — only agents, skills, hooks, and context.

## Quick Start

```
/dev-workflow <Work-Item-ID> [project-name] [team-name]
```

Defaults from `provider-config.md`. See the `/dev-workflow` skill for full workflow documentation.

## Workflow Phases

1. **Requirements Ingestion** — Planner pulls story/issue from configured provider, asks clarifying questions. *No feature branch exists yet.*
2. **Planning & Approval** — Planner proposes approaches, human selects one, plan + test outline + tracker generated — **GATE #1**
   - *Pre-flight runs immediately after GATE #1 clears* (`commands/preflight.md`): the orchestrator reads the tracker's `## Repo Status` section (populated by the Planner per `plan-generator/SKILL.md` Step 7), creates the `<team>/feature/<id>-<slug>` branch in **exactly** the repos the plan named (never in every known repo as a "safe default" — the prior behaviour produced orphan branches in unaffected repos), and commits the plan onto the new feature branch in the single-repo workspace-is-git-repo case. Pre-flight has no human gate.
3. **TDD Development Loop** — For each task: Tester writes failing tests → Developer makes them pass → Reviewer reviews combined diff → squash-merge
4. **Human Approval** — Human reviews full implementation — **GATE #2**
5. **Test Hardening** — Tester fills integration/E2E gaps, enforces ≥ 90% coverage, Reviewer reviews
   - *Static security review (P5.5) runs after Phase 5 completes* (`commands/security-review.md`): the orchestrator dispatches per-repo SAST lanes per `language-config.md`, aggregates findings, and stamps `Security review completed <ts>`. When any finding ≥ medium severity surfaces, **GATE #2.5** fires and the human picks `waive` / `fix-now` / `defer`. See `dev-workflow/SKILL.md` Commands table for invocation rules.
6. **PR Creation** — Reviewer does holistic pre-PR review of entire feature branch (all tasks combined, impl + tests, against plan + conventions) → detailed report shown to human → human approves — **GATE #3**
7. **PR Review Response** *(on-demand, after Phase 6)* — Reviewer challenges each PR comment against plan + acceptance criteria, classifies as VALID/INVALID/PARTIAL → findings report shown to human → human selects which comments to address — **GATE #4** → Planner adds new tasks → re-enters Phase 3 loop for those tasks. Repeatable across multiple review rounds.

**Inter-gate: Ad-Hoc Request Handling** *(on-demand, between any pair of gates)* — Human submits a change request at GATE #2 (instead of `APPROVED`), at GATE #3, or mid-phase via `/dev-workflow request <story-id> "<text>"`. Reviewer triages each request against the approved plan, classifies as IN_SCOPE_BUG / IN_SCOPE_AC_MISS / OUT_OF_SCOPE / PLAN_CONFLICT / DUPLICATE / INVALID → findings shown to human → human confirms in-scope items or chooses expand-scope / defer / withdraw for out-of-scope — **GATE #5** → Planner appends rows under `## Ad-hoc Tasks (Batch <N>)` → re-enters Phase 3 loop for those tasks. Repeatable across multiple batches. See `skills/dev-workflow/commands/handle-request.md`.

### Cross-cutting & utility commands

These run outside the linear 1→7 pipeline. Full invocation contracts live in [`skills/dev-workflow/SKILL.md`](skills/dev-workflow/SKILL.md) (Commands table).

- **P8 `reconcile`** — post-merge cleanup: marks the Story-State `Archived`, renames `tracker.md → tracker.archived.md`, stamps `Merge detected` and `Workflow completed`.
- **P9 `metrics`** — aggregates `Plan approved` / `PR created` / per-task durations into `metrics-report.md` + appends to `ai/_metrics-log.csv`. Auto-triggered at T1 (post-PR), T2 (each review cycle), T3 (post-reconcile); also runnable ad-hoc.
- **R `resume`** — workflow-state recovery after a crash; reads `Recovery-State:` and re-enters the interrupted phase. Pair: `/dev-workflow abort <id>` renames `tracker.md → tracker.aborted.md`.
- **`hotfix`** — re-entry on an `Archived` story; operates on a **clone** so the original `Archived` row is preserved.
- **`migrate`** — one-time `ai/plans/` + `ai/tasks/` → `ai/<YYYY-MM-DD>-<work-item-id>/` layout migration. Run once per workspace; the v2.0 startup gate refuses other commands when the legacy layout is detected.
- **`request`** — see Inter-Gate Ad-Hoc Request Handling above.

## Critical Ownership Rules

- The **Orchestrator** is the sole owner of the task tracker. It updates status after every agent verdict. The tracker stays **uncommitted** until Phase 6.
- The **Tester** (Phase 3) commits failing tests only — commit format `#<STORY-ID> #<TASK-ID> test: <slug>`. Never writes production code.
- The **Developer** commits production code only (no tracker, no test modifications). Works inside the orchestrator-created worktree (the orchestrator owns `git worktree add` in `commands/develop.md` Step 1 sub-step 5), or directly on the feature branch in the fallback path described in *Worktree Fallback* below.
- The **Reviewer** is strictly read-only — NEVER writes or edits any file. Returns its report to the orchestrator.

## Phase 3 Execution Order (NON-NEGOTIABLE)

### Within Each Repo (Sequential)

**For tasks marked `test-required: true` (default):**

1. Orchestrator updates tracker: T(n) → In Progress
2. Tester implements failing tests from the approved Test Outline in the worktree, commits `#<STORY> #T<n> test: <slug>`
3. Tester confirms new tests are **red** (fail for the right reason — not compile error)
4. Developer picks up the same worktree, runs tests (sees red), implements until all tests are green, commits `#<STORY> #T<n> impl: <slug>`
5. Orchestrator updates tracker: T(n) → In Review
6. Reviewer reviews the combined two-commit diff (tests + impl), returns verdict
7. Orchestrator handles verdict:
   - Approved → `git merge --squash` (both commits), tracker → Done, clean up worktree
   - Changes Requested → relay structured comments per the three-prefix model (`[R<n>]` → Developer, `[T<n>]` → Tester, `[S<n>]` → routed by file path) in the SAME worktree; when both impl and test comments exist, invoke Tester first then Developer; repeat from step 5

**For tasks marked `test-required: false`:**

1. Orchestrator updates tracker: T(n) → In Progress
2. Developer implements T(n) in worktree, commits code only
3. Orchestrator updates tracker: T(n) → In Review
4. Reviewer reviews worktree (read-only), returns verdict
5. Orchestrator handles verdict:
   - Approved → `git merge --squash`, tracker → Done, clean up worktree
   - Changes Requested → relay `[R<n>]` comments to Developer, fix in SAME worktree, repeat from step 3

**NEVER start T(n+1) in the same repo before Reviewer approves T(n).**
**NEVER squash-merge a worktree before Reviewer approves it.**
**NEVER have the Reviewer write or edit any file.**
**NEVER start the Developer on T(n) before the Tester commits failing tests for T(n), unless the task is `test-required: false`.**

### Across Repos (Parallel)

Repo lanes run fully in parallel via `run_in_background: true`. Repos never wait on each other. Cross-repo boundaries are resolved via **contracts** defined by the Planner in Phase 2.

## Legal Tracker Status Transitions

```
⏳ Pending       → 🔧 In Progress
🔧 In Progress   → 🔄 In Review
🔄 In Review     → ✅ Done           (reviewer approved)
🔄 In Review     → 🔧 In Progress    (changes requested)
✅ Done           → 🔧 In Progress    (rework)
```

Any other transition is blocked by the `tracker-transition-guard` hook.

## Non-Negotiable Rules

- Show a brief plan before taking action on any task. Wait for approval before executing.
- All commits: `#<STORY-ID> #<TASK-ID>: description` (both IDs mandatory; Task ID from planner e.g. T1, T2). TDD commits use `test:` or `impl:` suffix — `#<STORY> #T<n> test: <slug>` and `#<STORY> #T<n> impl: <slug>`. Every commit body must include `Co-Authored-By: Claude Code <noreply@anthropic.com>`.
- All branches: `<team>/feature/<id>-<slug>` — the middle segment is the literal string `feature`
- Build must pass the project's strictness policy as recorded in `language-config.md`. The harness warns at init-workspace time if no zero-warning enforcement mechanism is available for the detected language.
- Tests must achieve ≥ 90% line coverage on new/modified code only (test command per language-config.md). Do NOT go out of scope to cover pre-existing code. **Note**: `coverage-report` reports whole-file/package coverage — diff-aware filtering is not implemented. The Tester/Reviewer must inspect the per-file breakdown for files this story modified (via `git diff --name-only`) and confirm the threshold is met against those files, not the repo aggregate. See `coverage-report/SKILL.md` → *Scope — important*.
- Task tracker must be updated (in working tree) after every status change — committed once in Phase 6, amended in Phase 7
- Reviewer NEVER writes or edits files — orchestrator owns all tracker updates
- No code before plan approval. No PR before human approval.
- All agents must end responses with a `📋 AGENT STATUS` block
- **Sequential within each repo. Fully parallel across repos.** Cross-repo boundaries resolved via contracts.
- **One PR/MR per repo** in multi-repo stories — all linked to the same work item / issue
- **Every behavioral task begins with a failing test.** The Tester writes tests first (from the approved Test Outline); the Developer only makes them green. Tasks exempt from this are explicitly marked `test-required: false` in the plan.
- **Phase 7 is on-demand** — triggered by `/dev-workflow review-response <story-id>` after PR review comments arrive. It does NOT run automatically as part of the full pipeline.

## Provider Support

The workflow supports multiple providers via adapter skills in `skills/providers/`:

| Provider | Work Items | PRs / MRs | Configuration |
|----------|-----------|-----------|---------------|
| ADO | ✅ | ✅ | `provider-config.md` → `providers/ado/` |
| Jira | ✅ | Via git provider | `provider-config.md` → `providers/jira/` |
| GitLab | ✅ | ✅ (Merge Requests) | `provider-config.md` → `providers/gitlab/` |
| GitHub | ✅ | ✅ | `provider-config.md` → `providers/github/` |
| gh-cli | ❌ (git-only) | ✅ (Pull Requests via `gh` CLI) | `provider-config.md` → `providers/gh-cli/` |
| glab-cli | ❌ (git-only) | ✅ (Merge Requests via `glab` CLI) | `provider-config.md` → `providers/glab-cli/` |
| Zoho | ✅ (Mail Group Tasks) | Via git provider | `provider-config.md` → `providers/zoho/` |
| local-markdown | ✅ (local .md files) | Via git provider | `provider-config.md` → `providers/local-markdown/` |

Provider selection happens during `/init-workspace` and is stored in `.claude/context/provider-config.md`.

**Phase 7 (PR review-response) support** is declared per git provider in `skills/providers/<git-provider>/pr-comments.md` and covers all five git providers: GitHub (`gh api graphql` for review threads, REST replies), gh-cli (same CLI primitives), GitLab (REST `/discussions` via `curl`), glab-cli (`glab api` discussions), and ADO (REST `/threads` via `curl`). MCP wrappers exist for some providers and are marked 🟡 in each adapter — verify the tool surface in your harness install before relying on them. See `skills/providers/shared/capabilities.md` for the canonical capability list and sweep status.

## Worktree Fallback (Windows)

Git worktree creation may fail with `error: could not lock config file .git/config: File exists`. The orchestrator owns worktree creation in `commands/develop.md` Step 1 sub-step 5 — the Developer / Tester never see the failure. The flow is:

1. Orchestrator attempts `git worktree add <path> -b <branch> <feature-branch>`.
2. On failure, the orchestrator retries once (the lock error is usually transient).
3. On second failure, the orchestrator sets `worktree_failed: true` in the agent prompt's REPO_CTX block and the agent works directly on the feature branch.
4. The Phase 3 squash-merge (`commands/develop.md` Step 4 APPROVED) detects fallback mode and uses `git reset --soft <feature_head>` to collapse the agent commits, rather than `git merge --squash <worktree-branch>` (which would fail because no worktree branch exists). See the v2.0 CHANGELOG for the squash-merge fallback detail.

The agent never reports `Worktree: failed` — it reports `Worktree: not used (direct branch)` and `Worktree branch: n/a` per the schema in `agents/shared/status-schema.md`.

## Technology Stack

Per-repo language and framework configuration is discovered by `/init-workspace` and stored in `.claude/context/language-config.md`. Any language is supported — frontend and backend. The harness reads all build, test, coverage, and format commands from the generated context files.

---
> Source: [MostAshraf/ai-sdlc-harness](https://github.com/MostAshraf/ai-sdlc-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
