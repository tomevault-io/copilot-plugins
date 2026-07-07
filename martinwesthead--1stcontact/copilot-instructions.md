## 1stcontact

> provides: [batch_index, ...]

# Claude Instructions for 1stcontact

**Note:** This project uses XGD (Extreme Generative Development) methodology. XGD methodology docs are bundled in the xgd package (`xgd_source/system_docs/`) and loaded automatically into sessions.

## XGD Workflow Documentation

**System Documentation** (bundled in xgd package — `xgd_source/system_docs/` — read-only):
- `TDD-PROCESS.md` - TDD RED/GREEN workflow, quality gates, exception policy
- `TEST-STRATEGY.md` - Testing guidelines, what to test, parameterization
- `TASK-TEMPLATE.md` - Template for creating streamlined task prompts

These docs are bundled in the xgd package and loaded at runtime — not copied into your project.

## Branch Topology

**See `docs/reference/xgd-branching-strategy-full.md`** for the authoritative branch topology reference.

Summary:
- `xgd-working` — operator hot branch; all free-coded changes, tickets, config
- `main` — tested, reconciled truth; all branches cut from here
- `branch-<ticket>` — develop/feature branches; cut from main, merge back to main
- Reconcile branches — short-lived; promote working bundles into main
- Resync branches — short-lived; rebase working's unreconciled tail onto updated main
- Regression branches — cut from main; on success, fast-forward `xgd-stable`
- Merge branches — mechanical plumbing; created and deleted within a single run
- `xgd-stable` — production release; guaranteed conforming matrix and passing tests

## ⚠️ CRITICAL: No Legacy Modes or Backward Compatibility

**See also**: `PHILOSOPHY.md` (xgd system docs) → "Simplicity Over Preservation" and "Ruthless Refactoring"

**When refactoring, replace fully. Do NOT create legacy/fallback modes unless explicitly requested.**

- When implementing a new approach, replace the old one completely
- Do NOT add "legacy mode" or "fallback to old behavior" code paths
- Do NOT auto-detect "which mode to use" between old and new implementations
- Delete old code; don't comment it out or hide it behind flags
- Tests validate correctness, not legacy code paths
- Git history is the archive if old code is ever needed

**Why**: Legacy modes create:
- Complexity (two code paths to maintain)
- Confusion (which mode am I in?)
- Bugs (mode detection logic, edge cases)
- Dead code that never gets removed

**The Rule**: When replacing an approach, delete the old one. If tests pass, the new implementation is correct.

## ⚠️ CRITICAL: Failure vs Error Taxonomy (Workflow Outcomes)

**Failures, terminal failures, and errors are fundamentally different. Do NOT confuse them.**

| Category | What it is | Expected? | Recoverable? | System behavior |
|----------|-----------|-----------|--------------|-----------------|
| **Failure** | Tests fail, review fails, quality check fails | Yes | Yes (fix loop) | Retry via fix workflow (`@fail` → `fix_*` state) |
| **Terminal failure** | Guard not met, spec insufficient, validation gate | Yes | No | Graceful halt, clear message (`on_fail: terminal_failure`) |
| **Error** | Bug in system: missing FSM transition, broken config | **No** | **No** | **Immediate termination**, no handling, no cascading |

### The Distinction That Matters

- A **failure** is a workflow producing a negative result. The system has a defined path for it (fix loop).
- A **terminal failure** is an expected dead-end. The system stops gracefully.
- An **error** is a bug. The system is broken. There is no defined path because one should have existed but doesn't.

### Errors Must Not Be Handled As Failures

When the system encounters a bug (e.g., missing FSM transition), it MUST:
- **Terminate immediately** — do not cascade `@fail` through container hierarchy
- **Print a single clear error message** — not 3 cascading "failure" messages
- **Use distinct messaging** — "ERROR (BUG)" not "workflow failed"
- **NOT attempt recovery** — no transition, no retry, no fallback

### Tag Suffixes

- `@done` — workflow completed successfully
- `@pass` — gate/check passed
- `@fail` — workflow produced negative result (recoverable)
- `@loop` — iteration continues
- `@skipped` — skip condition met, workflow not needed
- `@error` — system bug, immediate termination (NOT a workflow outcome)

## ⚠️ CRITICAL: Prerequisite Semantics — Preconditions, NOT Flow Control

**Prerequisites are system preconditions. They are NOT workflow logic or flow control.**

A prerequisite checks that the system is in a valid state to run the workflow. If a prerequisite fails, it means there is a **bug** — missing data, broken config, corrupt state. The system MUST terminate immediately.

**Prerequisites MUST NOT:**
- Return `status: "done"` to signal "nothing to do" — that's flow control (use `skip_if` instead)
- Return `success: false` as a soft signal — it causes `FatalWorkflowError` → immediate termination
- Contain business logic about whether work remains — that belongs in `skip_if` or `exit_conditions`
- Be used to decide loop continuation — that's what `exit_conditions` with `loop`/`done` suffixes are for

**Prerequisites MUST:**
- Validate that required data exists (tickets, reports, config)
- Populate context variables needed by the workflow action
- Raise/fail immediately if the system state is invalid

**Anti-pattern (causes thrashing loops):**
```yaml
# ❌ WRONG: Prerequisite returns status="done" when no batches remain
# This signals "skipped" → loop breaks → parent re-checks → infinite loop
prerequisites:
  - kind: python
    function: get_next_batch  # Returns {"success": True, "status": "done"} when batch_total=0
    provides: [batch_index, ...]
```

**Correct pattern:**
```yaml
# ✅ RIGHT: skip_if handles "nothing to do", prerequisite only populates context
skip_if:
  done:
    NOT:
      ticket_exists:
        query: "type=report AND ... AND fields.batches_pending=true"

prerequisites:
  - kind: python
    function: get_next_batch  # Only called when skip_if confirms work exists
    provides: [batch_index, ...]
```

**Why this matters:** When a prerequisite returns `status: "done"`, the workflow is marked "skipped". In a loop state, "skipped" triggers `on_break` (same as "done"), which exits the loop back to the parent — which re-evaluates and re-enters the loop, causing infinite thrashing. The prerequisite was never meant to control flow; it was meant to fail-fast on bugs.

## Project Overview

(To be filled in with project-specific details)

## Core Principles

### Language & Platform
- (Specify language, frameworks, and platform)
- (e.g., Python 3.11+, Node.js 18+, Swift 5.9+, etc.)

### Architecture
- (Describe high-level architecture)
- (Module structure, key components, design patterns)

## Development Workflow

### XGD Workflow Constraints
**IMPORTANT:** The following constraints apply when using XGD (Extreme Generative Development) workflow:

- **DO NOT create git hooks** (pre-commit, pre-push, commit-msg, etc.)
  - Quality enforcement happens via `xgd quality run`
  - Git hooks create unrecoverable failure points that block workflow automation
  - All quality checks (lint, format, tests) are run before commits in the workflow pipeline

- **DO NOT modify .git/hooks/** directory
  - Let the XGD workflow manage quality gates
  - Quality configuration is auto-generated by `xgd quality config --apply`

- **DO NOT write to .xgd/status.json directly**
  - This file is managed by the XGD workflow engine
  - Only Python/Bash scripts in the workflow should modify it
  - Prompts should read it but never write to it

- **Tickets MUST be accessed exclusively through the ticketing API** (`xgd ticket` CLI or `xgd_source/core/ticketing` module)
  - NEVER read, write, or modify ticket `.md` files directly via filesystem operations
  - The file-based storage format is an internal implementation detail that may change
  - Direct file access bypasses locking, validation, indexing, and git commit

### Free-Coded Commit Convention

When committing bug fixes or improvements directly to main (outside of a development branch workflow), include `[FREE-CODED]` in the commit message:

```bash
git commit -m "fix(module): description of fix [FREE-CODED]"
```

**Why**: Long-lived branches don't have these fixes. When they merge back, the branch's older version can silently overwrite the fix. The merge review checks for `[FREE-CODED]` commits and flags overwrites as CRITICAL.

**`bin/project/xgd_version_bump` MUST be implemented before the free-coding gate allows progression.**
The gate command (`xgd ticket move-to-free-coded`) calls `bin/project/xgd_version_bump --check`
to verify the version bump is present in your commits. The stub exits 1 on bare invocation and
will block every ticket promotion until you implement the script. See the stub at
`bin/project/xgd_version_bump` for the required interface and implementation guide.

### Free-Coding Lifecycle (full process) — MANDATORY in user sessions

**Applies to interactive user sessions only.** Headless processes (`xgd develop`, `xgd reconcile`, `xgd regression`) do not follow this process — they have their own workflow state machines.

For any direct code change request in an interactive session ("implement X", "fix X", "add X"), the free-coding process is **mandatory** — not a guideline. Non-compliance has automatic consequences: commits without a scope ticket are detected and reverted on the next sync; stale ticket bodies corrupt the capability matrix when reconcile fires.

Full process and rules: **`FREE-CODING.md`** (bundled in xgd system docs, loaded automatically into every `xgd claude` session).

### Version Control Requirements
**CRITICAL:** .xgd/ state files MUST be tracked in version control for rollbacks to work correctly.

**DO track in git:**
- `.xgd/status.json` (workflow state)
- `.xgd/config.yaml` (project configuration)
- `.xgd/stage_prompts/` (stage-specific prompts)
- `.xgd/validation-reports/` (validation results)

**DO NOT track in git (already in .gitignore):**
- `.xgd/logs/` (regenerated on each run)
- `.xgd/tmp/` (temporary files)

**⚠️  NEVER add these patterns to .gitignore:**
- `.xgd/` (ignores everything - breaks rollbacks)
- `.xgd/*` (ignores everything - breaks rollbacks)
- `.xgd/**` (ignores everything - breaks rollbacks)

**Why this matters:** When reverting code via git, the .xgd state must also revert to maintain workspace consistency. If .xgd files are ignored, rolling back code leaves stale workflow state, causing failures.

### Quality Standards
- All code must pass lint checks with zero errors and zero warnings
- All tests must pass
- **Code coverage must meet the configured minimum threshold**
- Build must complete with no errors or warnings

**Coverage Enforcement:**
The coverage threshold is configured in `.xgd/config.yaml` under `quality.min_coverage_percent`.
- Default: 60%
- Project can override this value based on their quality requirements
- The quality check will fail if coverage is below the configured threshold

**DO NOT attempt to bypass coverage requirements by:**
- Modifying timeout values or other workarounds
- Proposing exceptions for coverage gaps (see Exception Policy below)

**The ONLY way to pass quality checks is to add sufficient tests to achieve the configured coverage threshold.**

## ⚠️ CRITICAL: Proactive Investigation — Do the Legwork

**The assistant's job is to absorb system complexity, not route it back to the user.**

Before asking the user for any information, check whether it can be retrieved from the system. If it can, retrieve it yourself.

### Retrievable Sources (check these before asking)

| Source | How to access |
|--------|--------------|
| Workflow logs | `.xgd/logs/*.jsonl` (parse as JSONL — do not ask user to read raw files) |
| Tickets | `xgd ticket get <uid>` or `xgd ticket list --type <type>` |
| Code / config | Read the files directly |
| Git state | `git log`, `git status`, `git tag -l`, `git worktree list` |
| Execution errors | Tail the most recent session log for the relevant context |

### When it's valid to ask the user

- The information is a business decision or intent not recorded anywhere in the system
- It requires human judgment that cannot be inferred
- You have genuinely searched the system and cannot find it

### Anti-pattern to avoid

❌ Asking "do you have the error message?" when the error is in the system logs.
✅ Find the relevant log, parse it, extract the error yourself.

**Why**: Users adopt XGD to reduce their burden. Routing system lookups back to the user undermines the core value proposition.

## Coding Guidelines

(To be filled in with project-specific coding standards, patterns, and best practices)

## Testing Strategy

### Test Suite Organization
**IMPORTANT:** Organize test suites by package/module/component, NOT by user story or feature.

**Correct approach:**
- One test suite per package/module (e.g., "Core Module Tests", "API Tests", "UI Tests")
- Groups related tests that share setup/teardown
- Minimizes redundant initialization (e.g., iOS Simulator boots once per suite, not per story)

**Incorrect approach:**
- One test suite per user story (e.g., "US001 Tests", "US002 Tests")
- Causes inefficient repeated initialization
- For iOS apps: Creates new simulator instance per suite (very slow)

### UAT-Primary Testing
**UATs are the primary validation layer.** Unit tests exist ONLY when UATs cannot cover complex internal logic.

- **UATs (3-6 per task)**: Test complete user workflows from entry point to success. One per acceptance criterion. PRIMARY.
- **Unit tests (0-3 per task)**: Only for complex algorithms with many internal branches. SUPPLEMENTARY.

**Coverage requirement (≥60% default)** should be met primarily through UAT coverage.
See `TEST-STRATEGY.md` (bundled in xgd system docs) for full testing guidelines.

### Test Timeouts
Configure timeouts based on test type:

- **Unit tests**: 10-30 seconds
- **Integration tests**: 60-120 seconds
- **iOS Simulator tests**: 300 seconds (5 minutes)
  - Simulator cold boot: 20-60s
  - App install: 5-10s
  - Test execution: varies
  - Coverage collection: 5-10s

**If tests consistently time out, investigate:**
- Hung tests (infinite loops, deadlocks)
- Missing test isolation (tests depending on each other)
- Inefficient test suite organization

(Additional project-specific testing approach, frameworks, and requirements to be filled in)

## Documentation Requirements

(To be filled in with documentation expectations)

---
> Source: [martinwesthead/1stcontact](https://github.com/martinwesthead/1stcontact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
