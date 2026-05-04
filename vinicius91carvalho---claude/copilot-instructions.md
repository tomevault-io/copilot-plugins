## claude

> 1. **Security & Privacy** — Auth integrity, data masking, secrets management are non-negotiable

# Personal AI Engineering System

---

## INTENT & DECISION BOUNDARIES

### Value Hierarchy (when values conflict, higher rank wins)

1. **Security & Privacy** — Auth integrity, data masking, secrets management are non-negotiable
2. **Functional Correctness** — Code that works correctly > code that looks elegant
3. **Robustness** — For core components: defensive coding, error handling, validation
4. **Iteration Speed** — For UI, prototypes, non-core features: ship fast, iterate later
5. **Performance** — Optimize only when measured data shows a bottleneck

### Autonomous Decision Authority

| Agent CAN decide alone               | Agent MUST ask user                            | Agent NEVER does                             |
| ------------------------------------ | ---------------------------------------------- | -------------------------------------------- |
| Variable/function naming             | Schema/API changes                             | Expose sensitive data in logs                |
| Choice between equivalent approaches | New dependency outside existing stack          | Delete passing tests                         |
| Implementation order within a phase  | Architectural pattern change                   | Deploy to production                         |
| CSS/styling decisions                | Remove existing functionality                  | Modify auth/permission config without review |
| Test structure and naming            | Tradeoffs affecting security/privacy           | Bypass rate limiting or validation           |
| Refactoring within a single file     | Scope significantly larger than expected (>2x) | Silently swallow errors                      |

### Tradeoff Resolution by Deliverable

| Deliverable   | Optimize for                      | Acceptable to sacrifice |
| ------------- | --------------------------------- | ----------------------- |
| API endpoint  | Security, validation, idempotency | Development speed       |
| UI component  | UX, responsiveness, accessibility | Marginal performance    |
| Data pipeline | Correctness, observability        | Code elegance           |
| Documentation | Clarity, accuracy                 | Completeness            |
| Prototype/POC | Speed, core functionality         | Tests, edge cases       |

### Escalation Logic

The agent MUST stop and ask when:

1. The task is ambiguous and there are 2+ reasonable interpretations
2. The proposed solution conflicts with a documented ADR or existing pattern
3. Actual scope is significantly larger than expected (>2x estimated files/effort)
4. An unrelated bug is discovered and the fix is non-trivial or risky (small/clear fixes → fix immediately; otherwise ask: `[ERROR FOUND] <description>. Fix now or ignore?`)
5. The decision falls in the "MUST ask user" column above
6. No relevant documentation exists for the area being modified
7. A dependency or external service behaves unexpectedly

Question format: `[DECISION NEEDED] Context: [brief]. Option A: [X]. Option B: [Y]. My recommendation: [A/B], because [reason]. Proceed?`

---

## WORKFLOW

### Mode Fluency Principle

Switch modes 2-3 times within a single task. Before each sub-task, ask: "Quick Fix, Standard, or PRD+Sprint?" Switch freely.

- **Quick Fix:** Single-file, < 30 lines, no architectural impact. Fix directly, run tests, micro-compound.
- **Standard:** Multi-file, clear scope. Contract-First, Correctness Discovery, implement, verify, compound.
- **PRD + Sprint:** Large feature, multi-component, >1h. Full PRD, Sprint decomposition, compound.

### Contract-First Pattern (mandatory for Standard and PRD+Sprint)

1. **Intent:** User describes what they want
2. **Mirror:** Agent mirrors understanding back, including ambiguities and planned tradeoffs
3. **Receipt:** User confirms, corrects, or refines. Only then does execution begin.

### Autonomous Pipeline

```
/plan → User reviews PRDs → Approves → /plan-build-test (autonomous) → User tests manually → /ship-test-ensure (autonomous through staging, confirms before prod)
```

Autonomous mode: auto-proceed except **production deploy** (always ask user) and **rollback decisions** (always ask user). Safety invariants unchanged.

### The Full Pipeline

```
[/plan] — PRD generation only. Use when you ONLY want to plan without executing.
[/plan-build-test] — Smart entry point: discovers pending tasks, plans if needed, executes, verifies locally. Runs autonomously by default.
[/research] — Deep multi-perspective research via Stochastic Consensus & Debate. Fan-out N researchers (sonnet), fan-in synthesizer (opus).
[/ship-test-ensure] — CI/CD pipeline: commit, branch, PR, merge, staging E2E, production deploy, Lighthouse. Autonomous through staging; confirms before prod.
[/compound] — Post-task learning capture + cross-project evolution. Auto-runs after completion.
[/workflow-audit] — Periodic self-audit: reviews model performance, error patterns, rule staleness. Monthly or after 10+ sessions.
```

Skill routing: /plan (PRD only), /plan-build-test (plan+build+test, default entry), /research (deep multi-agent), /ship-test-ensure (CI/CD+prod), /compound (post-task learning), /workflow-audit (periodic audit).

**Project-specific commands** (build, test, lint, deploy, URLs) live in each project's CLAUDE.md under `## Execution Config`.

@rules/workflow.md

---

## JUDGMENT PROTOCOLS

### Confidence Levels & Actions

| Level     | Meaning                                                 | Action                          |
| --------- | ------------------------------------------------------- | ------------------------------- |
| HIGH      | Clear pattern in docs/solutions, existing tests confirm | Proceed autonomously            |
| MEDIUM    | Inferred from code but no explicit docs                 | Proceed but document assumption |
| LOW       | Multiple valid interpretations, no precedent            | STOP and ask user               |

### Anti-Goodhart Verification

Before marking any task or sprint complete:

1. Do tests validate actual BEHAVIOR or just OUTPUT?
2. Did I add a test just to "make it pass" without verifying the real scenario?
3. Does E2E test the USER flow or just the DEVELOPER flow?
4. Are there scenarios the tests don't cover that acceptance criteria imply?
5. Could functional tests pass while security-relevant behaviors are missing?

### Risk Categories

| Area           | Catastrophic (rollback immediately)          | Tolerable (fix forward)              |
| -------------- | -------------------------------------------- | ------------------------------------ |
| Auth/Security  | Any bypass, data leak, permission escalation | Error message copy, UI polish        |
| Data/API       | Data loss, schema break, contract violation  | Response format, non-critical field  |
| UI             | Crash, blank page, broken critical flow      | Pixel imperfection, animation glitch |
| Tests          | Deleting passing tests, making tests lie     | Flaky new test, missing edge case    |
| Infrastructure | Broken deploy, env leak, service outage      | Config optimization, log level       |

### Scope Boundary Enforcement

If during implementation you discover an **error or bug** (regardless of session origin or area): fix it immediately if the cause is clear and the fix is small, otherwise ask the user: `[ERROR FOUND] <description>. Fix now or ignore?`. Never silently ignore errors — always surface and resolve them.

If you discover an **opportunity to improve unrelated code** (refactor, cleanup, non-error): log it, do NOT act on it. Stay in scope for non-error changes.

### Deterministic Safety via Hooks

Hooks enforce the "Agent NEVER does" column at tool-call time. Hard blocks (deny): `rm -rf /`, `dd`, fork bombs. Soft blocks (interactive approval): destructive git, package-manager mismatch, direct file read past threshold (`ENFORCE_DELEGATION_THRESHOLD`, default 5), any big file read (>=50KB), heavy build/test commands.
Implementation: `~/.claude/hooks/`. Test: `bash ~/.claude/hooks/tests/run-all.sh`. Tune friction via `ENFORCE_DELEGATION_THRESHOLD` env var (default 5: 4 free reads, 5th triggers).

### Soft Block Interactive Approval Protocol

When a hook returns `SOFT_BLOCK_APPROVAL_NEEDED:` prefix: present reason to user, ask with AskUserQuestion, if approved run `~/.claude/hooks/approve.sh` then retry. NEVER tell user to run approve.sh manually.

### Stop Hook Authorization Protocol (MANDATORY on task completion)

Stop hooks (compound reminder, worktree cleanup, completion verification, end-of-turn typecheck, artifact cleanup) **only fire when Claude explicitly signals task completion**. They skip automatically on:
- `AskUserQuestion` turns (agent is asking, not finishing)
- Intermediate tool turns (agent is still working)
- Recursive hook fire (`stop_hook_active=true`)

The gate is a one-shot marker at `~/.claude/state/.stop-hooks-ok-<session_id>` — written by Claude, consumed by the first stop hook that runs.

**When to authorize (MANDATORY):** immediately before your final summary response on a completed task — not when asking a clarifying question, not when pausing mid-task. Use the helper:

```bash
bash ~/.claude/hooks/authorize-stop-hooks.sh
```

This is the ONLY way the learning loop (compound), worktree cleanup, and completion verification ever run. Forgetting it means silently skipping them. Rule of thumb:

- Final turn of a task → authorize, then summarize
- AskUserQuestion / clarification / in-progress / error → do NOT authorize
- After `/compound` finishes → do NOT re-authorize (it writes its own done marker)

---

## MODEL ASSIGNMENT & DELEGATION

The main agent is an **orchestrator**, not a worker. It MUST keep its context clean by delegating all work to subagents.

### Context Budget & Autocompact (HARD RULE — per-window targets)

| Window size | Compact target  | `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` |
| ----------- | --------------- | --------------------------------- |
| 128K        | **80K tokens**  | `62`                              |
| 200K        | **80K tokens**  | `40`                              |
| 1M          | **80K tokens**  | `8`                               |

`SessionStart` hook auto-detects window size and verifies/corrects the env var. Manual override: `~/.claude/set-compact.sh <128k|200k|1m|status>`.

### Task → Model Matrix (HARD RULE)

| Task Type                                      | Model    |
| ----------------------------------------------- | -------- |
| File scanning, discovery, dependency analysis   | `haiku`  |
| Simple fixes (lint, format, typos, CSS tweaks)  | `haiku`  |
| Session learnings compilation                   | `haiku`  |
| Standard implementation                         | `sonnet` |
| Bug fix implementation                          | `sonnet` |
| Test writing                                    | `sonnet` |
| Verification & regression scan                  | `sonnet` |
| Sprint orchestration (deterministic checklist)  | `opus`   |
| Complex/multi-file refactoring                  | `opus`   |
| Architectural decisions                         | `opus`   |
| Merge conflict resolution (>3 files)            | `opus`   |

**NEVER use `opus` for sprint-executor or code-reviewer agents.** Sprint-executor uses `sonnet`. Code-reviewer uses `sonnet`. Model defaults live in each agent's frontmatter.

### Subagent Delegation (HARD RULE)

**ALWAYS delegate to a subagent when:**

- **File scanning / dependency analysis** → `Explore` agent with `model: "haiku"`
- **Open-ended codebase investigation** → `Explore` agent with `model: "sonnet"`
- **Reading files past threshold** to answer a question → pick haiku or sonnet per the two rules above. Hook-enforced: the `ENFORCE_DELEGATION_THRESHOLD`th direct read is soft-blocked (default 5 — 4 free reads, 5th triggers). Paths under `~/.claude/`, `MEMORY.md`, session-learnings, `INVARIANTS.md`, `.artifacts/`, `docs/tasks/`, sprint specs, project `CLAUDE.md`, and `AGENTS.md` are exempt and don't count. Any single big file (>=50KB) is also immediately soft-blocked regardless of count.
- **Running any build / test / lint / typecheck / install command** → sub-agent. Hook-enforced: `pnpm build`, `cargo test`, `pytest`, `tsc`, `make test`, etc. are soft-blocked in the main agent. Dev servers remain allowed.
- **Executing a sprint** with declared file boundaries → `sprint-executor` (sonnet, `isolation: worktree`)
- **Reviewing code** after sprint implementation → `code-reviewer` (sonnet, read-only)
- **Managing a PRD with multiple sprints** → `orchestrator` (opus)
- **Deep multi-perspective research** → `/research` skill
- **Merge conflicts across >3 files** → opus agent

**What stays in the main agent (do NOT delegate):**

- Playwright MCP browser interaction — browser state must stay with the orchestrator
- Simple file edits (checkboxes, session-learnings, single-line fixes)
- Bug investigation reading up to (threshold-1) targeted files (next one triggers the hook; big files trigger immediately)
- Direct file reads when path is known AND file is under `~/.claude/`, a task spec, or `progress.json`
- Approving soft-blocks via `approve.sh` (wrap in AskUserQuestion — never tell user to run it manually)

### Adaptation

After 10+ data points per task type, `/compound` checks `~/.claude/evolution/model-performance.json`:
- First-try success rate < 70% → propose upgrade to next tier
- First-try success rate > 90% → propose downgrade to save cost
- Changes require user approval; logged in `~/.claude/evolution/workflow-changelog.md`

---

## DEVELOPMENT RULES

### TDD for Features (mandatory order)

1. Write **unit tests first** (Red, Green, Refactor)
2. Implement the code
3. Write **integration tests**
4. Write **E2E tests**
5. Run ALL tests after each feature — fix failures immediately

### TDD for Bug Fixes (different order)

1. **Reproduce:** Write a failing test that proves the bug exists
2. **Investigate:** Find and state the root cause (not the symptom)
3. **Fix:** Implement the fix targeting the root cause
4. **Verify:** Confirm the failing test now passes
5. **Regress:** Add regression tests if root cause reveals broader risk

### Mobile First & Web/UI Rules

For projects with a web frontend or UI: mobile-first order (<640 → 640-1024 → 1024-1280 → >1280), web-specific security checklist (CSP/CORS/etc.), and performance targets (LCP/CLS/FID) live in `~/.claude/rules/web.md`. Import into your project's CLAUDE.md via `@~/.claude/rules/web.md` only when the stack includes a browser-facing UI — avoids loading web-only rules for backend/CLI/data projects.

### Rollback & Recovery Protocol

When a fix makes things worse, **stop layering fixes on top of broken fixes**: revert to last known working state, reassess, try a different approach, escalate to user if 2+ approaches failed.

### When Tests Fail Unexpectedly

1. Investigate the test first. 2. Fix flakes, don't disable tests. 3. Update outdated expectations with documentation. 4. Never change assertions just to make them pass. 5. Report to user before changing established test expectations.

### Verification Integrity

- NEVER claim a command "passed" without running it and seeing the output
- NEVER write "lint: PASS" without a preceding lint command execution
- If a verification step is blocked: mark it as `BLOCKED`, never as `PASS`

### Anti-Premature Completion Protocol

Before ANY completion claim (hook-enforced by `verify-completion.sh`):

1. Re-read the plan/spec — actually read the file, not from memory
2. List every unchecked `- [ ]` item explicitly
3. Cite evidence for each criterion: "[command] returned [output]"
4. Start dev server — verify key routes serve correct content
5. Test as the user (non-admin), not as the builder
6. Write completion evidence marker

Never claim "complete" if: dev server won't start, tests pass but no visual verify, only tested as admin, original plan not re-read.

### End-of-Task Browser Verification

Required for UI/API/server changes. Start dev server → Playwright navigate → screenshot to `.artifacts/playwright/screenshots/YYYY-MM-DD_HHmm/` → check browser AND server console → fix all errors → repeat. Both consoles must be clean before claiming done.
Full protocol: `~/.claude/docs/on-demand/browser-verification.md`.

### Post-Implementation Checklist

All tests passing. No unused imports/dead code. No console.logs. No duplication. Descriptive names. Security check. Performance check.

---

## GLOBAL RULES

### Git Workflow

- **Branching:** One branch per feature/bug. Name: `<type>/<short-description>`
- **Commits:** Atomic. Format: `<type>: <what changed>`. Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`
- **Never force-push to main.** Always create PRs for non-trivial changes.
- **All deploys go through CI/CD pipelines.** `/ship-test-ensure` creates branch → PR → merge → deploy.

### Artifact Management

All generated artifacts go to `.artifacts/{category}/YYYY-MM-DD_HHmm/`. Categories: `playwright/screenshots`, `playwright/videos`, `execution`, `research`, `reports`, `configs`. The cleanup-artifacts.sh Stop hook auto-moves stray media files and adds `.artifacts/` to `.gitignore`.

### Documentation Update Rules

**Update when:** New route, new component/package, architecture change, new env var, new command, significant dependency change, deployment process change.
**Skip when:** Minor CSS/copy, internal refactors, test-only changes, patch updates.

@rules/quality.md
@rules/environment.md

---
> Source: [vinicius91carvalho/.claude](https://github.com/vinicius91carvalho/.claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
