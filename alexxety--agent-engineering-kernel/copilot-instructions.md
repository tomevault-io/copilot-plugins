## agent-engineering-kernel

> Bootstrap or audit a serious agent-led engineering workflow in any repository. Use when Codex must establish or update project canon, PRD-first execution, GitHub Epic/Task/Bug decomposition, PR verification discipline, multi-agent release coordination, reusable bootstrap files, or a model-agnostic workflow that future GPT/Codex and Claude-style agents can reuse without re-explaining the process in chat.


# Engineering Kernel

Use this skill when the task is to establish or upgrade a repository's engineering operating system, not when the task is simply to ship a normal feature.

Read [ENGINEERING_KERNEL.yaml](ENGINEERING_KERNEL.yaml) for the machine-readable core.
Read [references/BOOTSTRAP.md](references/BOOTSTRAP.md) when you need to materialize the kernel into a project.
Read [references/MODEL_ADAPTERS.md](references/MODEL_ADAPTERS.md) when the user asks how the kernel should map across GPT/Codex and Claude-style agents.
Read [references/BEHAVIORAL_OVERLAY.md](references/BEHAVIORAL_OVERLAY.md) when the user asks whether a thin behavior-only guideline layer should exist across `CLAUDE.md`, Cursor rules, or skill/plugin surfaces.
Read [references/SUPERPOWERS_SKILL_ORCHESTRATION.md](references/SUPERPOWERS_SKILL_ORCHESTRATION.md) when the user asks how Superpowers or another process-skill pack should be used with the kernel.
Read [references/AGENTIC_CODING_ORCHESTRATION.md](references/AGENTIC_CODING_ORCHESTRATION.md) when the user asks how orchestrator/worker/reviewer agentic coding should be structured, limited, reviewed, and recovered after resource failures.
Read [references/MULTI_AGENT_RELEASE_COORDINATION.md](references/MULTI_AGENT_RELEASE_COORDINATION.md) when the user asks how multiple agents, branches, worktrees, integration PRs, staging/prod deploys, runtime recovery, or handoffs should be coordinated in one production-bound project.
Read [references/PROJECT_LOCAL_WORKERS.md](references/PROJECT_LOCAL_WORKERS.md) when the user asks how to create global fallback workers, project-local workers, no-MCP coding/review/docs workers, rich-MCP orchestrators, or worker selection rules.
Read [references/MCP_TOOLING.md](references/MCP_TOOLING.md) when the user asks how MCP servers, App connector tooling, GitHub App permissions, or local `gh` fallback should be used.
Read [references/RESEARCH_POLICY.md](references/RESEARCH_POLICY.md) when the user asks how research and evidence collection should work.
Read [references/EXECUTION_SURFACES.md](references/EXECUTION_SURFACES.md) when the user asks where work should run locally versus in GitHub Actions or CI.
Read [references/ENVIRONMENT_PROMOTION.md](references/ENVIRONMENT_PROMOTION.md) when the user asks how local, verify, staging, and production environments should be separated and promoted safely.
Read [references/KERNEL_SYNC_POLICY.md](references/KERNEL_SYNC_POLICY.md) when the user asks how live project learnings should be reviewed and promoted back into the universal kernel.
Read [references/SESSION_ISSUE_SYNC.md](references/SESSION_ISSUE_SYNC.md) when the user asks how serious slices should update or explicitly skip GitHub issue state at closeout.
Read [references/WORK_ITEM_ROUTING.md](references/WORK_ITEM_ROUTING.md) when the user asks where work items, issues, backlog tasks, or cross-repo follow-ups should live.
Read [references/PROJECT_HEALTH_AUDIT.md](references/PROJECT_HEALTH_AUDIT.md) when the user asks how to audit a project for missing operating artifacts such as SSOT, decision log, escalation rules, incident log, or eval cases.
Read [references/OPTIONAL_WEEKLY_OPERATING_LOOP.md](references/OPTIONAL_WEEKLY_OPERATING_LOOP.md) when the user asks how optional weekly planning, outcomes, retro scorecards, or carryover decisions should work.
Read [references/KERNEL_UPSTREAM_AWARENESS.md](references/KERNEL_UPSTREAM_AWARENESS.md) when the user asks how consumer projects should notice upstream kernel changes and decide whether to adopt them.
Read [references/KERNEL_ADOPTION_TASK.md](references/KERNEL_ADOPTION_TASK.md) when the user asks what exact downstream `Task` should be opened or updated after `kernel_upstream_check` reports drift.
Read [references/KERNEL_FLEET_SWEEP.md](references/KERNEL_FLEET_SWEEP.md) when the user asks how one operator machine should check kernel drift across many consumer repositories at once.
Read [references/GITHUB_DELIVERY.md](references/GITHUB_DELIVERY.md) when the user asks how branch/PR/merge flow should work end-to-end.
Read [references/BUG_INTAKE.md](references/BUG_INTAKE.md) when the user asks how runtime failures should become GitHub `Bug` issues without noisy duplication.

The bug-intake rule is explicit:

- do not create bug issues from raw logs or chat alerts
- use one stable fingerprint per bug class
- update the existing open bug issue when the fingerprint matches

The session issue sync rule is explicit:

- serious closeouts record `Issue Sync: updated | skipped | not_applicable`
- durable status, next steps, verification, and links belong in the issue body
- comments are for short chronological notes, explicit user requests, or external blockers

The operational canon rules are explicit:

- `Work Item Routing` decides the target repo or project-local surface before issue creation; never use the current checkout as the implicit target
- `Project Health Audit` surfaces missing SSOT, metric definitions, freshness policy, decision log, escalation rules, incident log, prohibited actions, eval or golden cases, and critical runbooks
- `Optional Weekly Operating Loop` is not bootstrap default; when enabled, it plans outcomes, checks evidence, and gives stale carryover a terminal decision

The minimum Superpowers mapping is explicit:

- `using-superpowers` and `brainstorming` for new behavior
- `writing-plans` for approved multi-step work
- `test-driven-development` and `systematic-debugging` for implementation and bugs
- `verification-before-completion` before success or PR-ready claims

The agentic coding orchestration rule is explicit:

- one orchestrator remains accountable for scope, verification, docs, commits, and runtime checks
- rich-MCP orchestrators should dispatch project-local no-MCP coding/review/docs workers when custom agents are available
- project-local workers are preferred over the global no-MCP fallback for project implementation
- Claude Code can be used as a one-shot `design_readonly` or `review_readonly` adapter with direct-binary preference, `claude-opus-4-8` model pin, `stream-json --verbose`, explicit empty MCP config, strict MCP enforcement, `dontAsk` permission mode, mode-specific tools such as `Read` or `Read,Grep,Glob`, and budget caps
- Claude Code is high leverage for UX/workflow architecture, PRD/spec/runbook cleanup, independent plan/diff review, test planning, and refactor-boundary advice; prefer context packets, focused snippets, or selected diff context before large/whole files; `implementation_no_mcp` and `research_mcp_readonly` require a separate active PRD
- when the operator explicitly asks for Claude Code and it fails, diagnose Claude Code CLI/auth/MCP/tool/process setup instead of silently switching to GPT/Codex
- default to one worker agent at a time
- use at most two parallel worker agents only with disjoint write sets and healthy local executor state
- close completed subagent threads promptly
- keep implementation workers open only for the same-patch review/fix loop
- treat reviewer/explorer/docs-specialist threads as one-shot and use fresh reviewers for re-review
- require subagent final responses to include `thread_disposition`
- avoid parallel shell/tool calls while worker agents are active
- serialize git ref/index/worktree-mutating commands such as `git fetch`, `git pull`, `git switch`, `git merge`, branch deletion, and `git push` per repository; never run them in parallel for the same repo
- stop spawning agents and recover sequentially after executor/resource failures such as `Too many open files`
- resume only after `git status --short --branch` and `git diff --check` can run

The multi-agent release coordination rule is explicit:

- `main` is source of truth only after the verified integration PR is merged
- before merge, the source of truth is the active release candidate branch,
  commit SHA, PR, deploy command, runtime state, and verification evidence
- the repository's primary/root checkout is a coordination surface, not the
  default workspace for agent edits; every task uses an isolated worktree unless
  the handoff explicitly names the root checkout as the active worktree
- docs-only, PRD-only, and canon-only slices still use isolated worktrees when
  parallel work exists in the same repository
- never recover production by deploying from a stale, detached, or dirty
  checkout without first identifying the release candidate
- multiple agent branches that must ship together go through an integration
  branch, draft PR, staging verification, production verification when
  applicable, PR evidence, merge to `main`, and branch cleanup
- second agents touching live runtime must receive or reconstruct a handoff
  packet before acting

The MCP tooling rule is explicit:

- prefer MCP or App connector tooling for supported structured external-service operations
- local `gh` auth and GitHub App connector auth are different identities with separate permissions
- GitHub App repository access and permissions live in GitHub Installed Apps, not in the local `gh` token
- use shell-safe `gh` fallback when the connector is missing or blocked

The environment-promotion rule is explicit:

- production secrets and production database URLs must not be used in local config or local tests
- mutating automated tests must not run against production

The cutover entitlement parity rule is explicit:

- a new shell, navigation model, admin information architecture, or major UI cannot become default until role-matrix evidence proves the same entitlement source of truth is respected
- the role-matrix covers unauthenticated, personal, workspace owner/admin, workspace member, enterprise/admin, and platform admin surfaces
- desktop navigation, mobile navigation, command/search palette, topbar/page chrome, and direct restricted routes are checked before default cutover

## Use this skill for

- creating a reusable engineering kernel for future projects
- bootstrapping a new repository so agents stop depending on chat memory
- establishing PRD-first execution
- establishing Superpowers-compatible process-skill orchestration
- establishing orchestrator / worker / reviewer agentic coding limits and recovery protocol
- establishing multi-agent release coordination across branches, worktrees,
  integration PRs, staging, production, and runtime recovery
- establishing Claude Code as a bounded no-MCP subagent adapter
- establishing MCP/App connector tooling boundaries and GitHub App permission checks
- establishing GitHub `Epic / Task / Bug` workflow
- establishing work item routing before issue creation
- establishing project health audit checks for missing operational artifacts
- establishing optional weekly operating loops with outcomes and terminal carryover decisions
- establishing automatic deduplicated GitHub bug intake
- establishing Tavily Search-first research behavior, with Tavily Research reserved for justified expensive deep-sweeps
- establishing local-first execution and prerequisite bootstrap
- establishing environment promotion and production safety rules
- establishing cutover entitlement parity before a new shell/navigation/admin UI becomes default
- establishing kernel sync review and kernel impact discipline
- establishing session issue sync closeout discipline
- establishing kernel upstream awareness and downstream adoption checks
- establishing the canonical `kernel_adoption_task` work item for downstream kernel updates
- establishing optional multi-repo kernel fleet sweep on the operator machine
- running the `kernel_upstream_check` protocol in consumer repositories
- running the `kernel_fleet_sweep` protocol for multiple consumer repositories
- running the `kernel_sync_review` closure protocol
- establishing branch / draft PR / merge discipline
- establishing safe GitHub sub-issue linkage from normal issue numbers
- creating project-local canon files and templates
- creating repo-managed GitHub metadata and community-health files
- auditing an existing repo against the kernel and closing the gaps

## Do not use this skill for

- ordinary feature implementation
- routine debugging
- generic documentation edits
- code review that does not change the workflow canon

## Workflow

1. Inspect project reality first.
- source-of-truth docs
- runtime and CI entrypoints
- current GitHub workflow state
- current validation layers

2. Apply the minimal core before discussing maximum enforcement.
- project canon
- PRD-first execution
- Superpowers-compatible skill orchestration when process skills are available
- issue tree
- local-first execution surface
- MCP/App connector tooling policy
- environment promotion surface
- kernel upstream awareness
- kernel sync review
- PR verification contract
- ownership and labels
- research policy
- GitHub delivery flow
- work item routing
- project health audit
- optional weekly operating loop when the project chooses a weekly cadence

3. Keep the core model-agnostic.
One engineering process, thin adapters only.

4. Prefer deterministic bootstrap.
Use the provided templates and bootstrap script instead of recreating the same files from scratch.

5. Record plan-gated enforcement honestly.
If GitHub settings or plan limits block a stronger layer, document the gap. Do not pretend enforcement exists when the platform is not enforcing it.

## Output standard

Prefer a small durable set of outputs:

- project-local canon docs
- `AGENTS.md`
- `CONTRIBUTING.md`
- issue templates
- PR template
- labels source-of-truth
- `scripts/sync_github_labels.py`
- `CODE_OF_CONDUCT.md`
- `SECURITY.md`
- `CODEOWNERS`
- active PRD / decision record
- explicit bug-intake policy for verifier/watchdog/runtime incidents
- explicit environment-promotion policy for local, verify, staging, and production
- explicit `Issue Sync` closeout decision for serious slices
- explicit `Work Item Routing` decision when durable work could belong to another repo or surface
- explicit `Project Health Audit` findings for missing operating artifacts
- explicit `Optional Weekly Operating Loop` adoption or non-adoption when weekly cadence is discussed
- explicit `kernel_impact` field in PRD/closeout flow
- explicit `Kernel Impact` closeout decision after serious slices
- explicit process-skill policy for Superpowers or equivalent skills
- explicit agentic coding orchestration policy for worker/reviewer concurrency and resource recovery
- explicit project-local worker policy for rich-MCP orchestrators and no-MCP implementation workers
- explicit MCP/App connector policy with local `gh` fallback boundaries

Use the templates in [templates/project](templates/project) when bootstrapping a repo.

---
> Source: [alexxety/agent-engineering-kernel](https://github.com/alexxety/agent-engineering-kernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
