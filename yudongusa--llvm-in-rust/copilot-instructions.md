## llvm-in-rust

> This file documents the agentic workflow used to develop LLVM-in-Rust with

# AGENTS.md — Agentic Development Guide

This file documents the agentic workflow used to develop LLVM-in-Rust with
coding agents.  It exists so that agents can operate **autonomously** on this
project with minimal back-and-forth, following the patterns established across
the production-readiness roadmap (issue #93, Milestones A–Z).

---

## Standing Roadmap-Execution Workflow

This is the durable process for advancing the production-readiness roadmap
(**issue #93**).  It is the default mode of work — agents should follow it
without waiting for a per-task restatement of these rules.

1. **Source of truth is #93.** Pick up the lowest-lettered milestone that is
   still open (its tracking issue is linked from #93).  Milestones are worked
   roughly in order; later milestones may depend on earlier ones being green
   (e.g. the RC burn-in milestone is blocked until all prior milestones pass).
2. **One issue → its own focused PR(s).** Solve each tracking issue (or each
   self-contained checklist item within a large milestone) in a separate PR.
   Keep mechanical changes (fmt, clippy, renames) in their own PRs, separate
   from behavioral changes, so each diff stays reviewable.
3. **Every PR carries thorough tests.** Add regression tests that would have
   caught the bug or that exercise the new behavior.  Prefer
   target-independent tests (assert on IR / data structures) so they run on
   every platform, not just the host arch.
4. **Code review is mandatory** (see the loop below).  When two agents
   collaborate, divide milestones explicitly up front so branches never
   overlap, and review each other's PRs.  GitHub blocks self-approval when
   pushes share one account, so post the review as a PR **comment** with an
   explicit verdict instead of a formal approval.
5. **Quality gates must stay green** on the merge commit: `cargo +stable fmt
   --all -- --check`, `cargo +stable clippy --locked --all-targets -- -D
   warnings` (both enforced by `.github/workflows/quality-gates.yml`), the full
   test suite, the Differential Tests workflow, and the Platform Matrix gate.
   Do not mark any production-readiness status green unless all of these agree
   on the same commit.
6. **Close the issue after its PR merges** and tick the corresponding checkbox
   in #93; keep the #93 status table current.
7. **Report once a milestone is done** in the active coordination thread and
   update #93 with links to the green CI runs.
8. **Auto-resume across interruptions.** Keep `MEMORY.md` (in the agent
   workspace) current as the recovery point: which milestone is in flight,
   which PRs are open, and what is next.  After a rate-limit, sleep, or daemon
   restart, re-sync `origin/main`, re-read #93, and resume from the lowest open
   milestone — no human nudge required.

---

## Development Lifecycle

Every feature follows this six-stage cycle, executed end-to-end without
user prompts at each step:

```
Plan → Implement → PR Review → Test → Issue+Fix Loop → Merge
```

| Stage | Slash skill | Description |
|-------|-------------|-------------|
| Implement a phase | `/implement-phase` | Branch → code → targeted tests → commit → PR |
| Review implementation PR | `/review-and-fix` | Review diff/tests → run full tests → post PR feedback |
| Fix one issue | `/fix-issue <N>` | Read issue → fix in same PR branch → test → update PR |

### Mandatory PR Review/Test/Issue Loop (for implementation PRs)

Before merging an implementation PR, the agent must:

1. Review the PR diff and changed tests with a code-review mindset (correctness, regressions, missing tests).
2. Run targeted tests plus a full test sweep (`cargo +stable test` unless blocked).
3. If concrete problems are found, open GitHub issue(s) documenting them.
4. Fix those problems in the **same PR branch** and push follow-up commits.
5. Post PR feedback (`gh pr review --comment` or `gh pr comment`) summarizing findings, linked issues, and fixes.
6. Merge only when checks are green and no unresolved review findings remain.
7. After merging, verify and close the associated GitHub issue(s) referenced by the PR.

---

## Git Workflow Conventions

These rules prevent common mistakes in the multi-worktree setup:

| Rule | Reason |
|------|--------|
| Always branch from `origin/main`, not `main` | `main` is checked out in the primary worktree; checking it out again fails |
| `gh pr merge <N> --squash` (no `--delete-branch`) | Same worktree conflict |
| Stage specific files, never `git add -A` | Avoids accidentally committing `target/` or secret files |
| Never use `--no-verify` | Fix the hook failure instead |
| Run `cargo test` before every commit | All tests must be green |
| If review finds bugs, open issue(s) and fix them in the same PR branch | Preserves traceability and keeps context in one PR |
| Post at least one PR review feedback comment before merge | Captures reviewer reasoning and findings in GitHub history |
| Always verify and close associated issue(s) after merging a PR | Keeps GitHub project state aligned with merged work |

**Branch naming:**
- Features: `feature/phase<N>-<slug>` (e.g. `feature/phase4-x86-backend`)
- Fixes: `fix/issue-<N>-<slug>` (e.g. `fix/issue-30-mov-to-preg`)

---

## Agent Usage Guide

### rust-stable-compat agent
Use for issue #90 and any nightly-to-stable migration work.

```
Invoke: $rust-stable-compat
When:   Removing `#![feature(...)]`, migrating benches to stable,
        updating CI/docs, and validating stable build/test commands.
Skill:  skills/rust-stable-compat/SKILL.md
```

### simd-vectorization agent
Use for issue #86 and any x86 SIMD vector-lowering work.

```
Invoke: $simd-vectorization
When:   Adding vector IR lowering, SSE4.2/AVX2 emission, and target-feature
        gating in the x86 backend.
Skill:  skills/simd-vectorization/SKILL.md
```

### ipa-optimizer agent
Use for issue #87 and inter-procedural optimization work.

```
Invoke: $ipa-optimizer
When:   Building call-graph analysis, IPCP/dead-argument module passes, and
        integrating IPA into O3.
Skill:  skills/ipa-optimizer/SKILL.md
```

### riscv-backend agent
Use for issue #89 and RV64GC backend implementation work.

```
Invoke: $riscv-backend
When:   Adding the `llvm-target-riscv` crate, implementing regs/ABI/lowering/
        encoding, and validating RISC-V object generation tests.
Skill:  skills/riscv-backend/SKILL.md
```

### linker-compat agent
Use for issue #91 and linker/debugger compatibility validation work.

```
Invoke: $linker-compat
When:   Adding linker/tool integration tests, fixing ELF/Mach-O object
        conformance issues, and documenting exact link commands.
Skill:  skills/linker-compat/SKILL.md
```

### dwarf-debug agent
Use for issue #92 and DWARF debug metadata/line-table implementation work.

```
Invoke: $dwarf-debug
When:   Threading `!dbg` metadata through parser/codegen, emitting
        `.debug_line`, and validating debug output with toolchain utilities.
Skill:  skills/dwarf-debug/SKILL.md
```

### mem2reg-verification agent
Use for issue #83 and mem2reg formal/semantic verification work.

```
Invoke: $mem2reg-verification
When:   Adding mem2reg correctness invariants, Alive2 before/after corpus,
        and property-based semantic-equivalence tests.
Skill:  skills/mem2reg-verification/SKILL.md
```

### windows-debug-pdb agent
Use for issue #133 and Windows debug info pipeline work.

```
Invoke: $windows-debug-pdb
When:   Adding COFF object emission, CodeView `.debug$S` milestones, and
        Windows/PDB validation documentation/tests.
Skill:  skills/windows-debug-pdb/SKILL.md
```

### constant-folding agent
Use for issue #140 and middle-end constant-folding pass work.

```
Invoke: $constant-folding
When:   Adding dedicated compile-time constant evaluation pass behavior,
        pipeline integration at O1+, and fold/non-fold regression tests.
Skill:  skills/constant-folding/SKILL.md
```

### integrated-assembler agent
Use for issue #141 and direct MC/integrated assembler work.

```
Invoke: $integrated-assembler
When:   Formalizing explicit machine-IR -> object-byte assembly APIs, adding
        MC-stage docs/bench coverage, and preserving object-format correctness.
Skill:  skills/integrated-assembler/SKILL.md
```

### fp-memory agent
Use for Milestone I sub-issues: floating-point lowering and non-promotable memory.

```
Invoke: $fp-memory
When:   Implementing SSE2/NEON/RV-FD instruction lowering & encoding, XMM/V-register
        class support, FP calling convention, or stack-frame alloca/load/store
        with SP-relative addressing in any backend.
Skill:  skills/fp-memory/SKILL.md
```

### Plan agent
Use **before** starting a new phase or a non-trivial fix.

```
Invoke: Agent tool with subagent_type="Plan"
When:   Implementing a new crate, designing a data structure, or planning
        multiple-file changes across >3 files.
Output: Step-by-step plan written to /Users/yudong/.claude/plans/<name>.md
        followed by ExitPlanMode.
```

### Explore agent
Use for **codebase searches** when the location of something is unknown.

```
Invoke: Agent tool with subagent_type="Explore"
When:   Looking for where a trait is implemented, all uses of a type,
        or understanding how an existing subsystem works.
Levels: "quick" (single grep), "medium" (several files), "very thorough" (deep)
```

### general-purpose agent (background)
Use for **parallel independent work** — e.g. running tests in the background
while writing another file, or fetching GitHub issue data while reading source.

```
Invoke: Agent tool with subagent_type="general-purpose", run_in_background=true
When:   The sub-task is independent of the current work and would block
        the main thread if run synchronously.
```

### When NOT to use agents
- Reading a specific known file → use `Read` directly
- Searching for a specific class or function → use `Grep` directly
- Simple one-file edits → use `Edit` directly

---

## Autonomous Milestone Execution (standing instruction)

When the user asks to "work on remaining milestones" or "finish the roadmap",
Claude must execute the following loop **without waiting for user prompts**
at each step:

1. Read `MEMORY.md` and `#93` to determine which milestones and sub-issues are open.
2. For each open milestone, lowest letter first (consult #93 for the current
   open set — e.g. Milestones V → W → X → Y → Z after the 2026-05 audit):
   a. Work on each sub-issue in dependency order (infrastructure first).
   b. Use parallel worktree agents for independent sub-issues within a milestone.
   c. Follow the full PR workflow (implement → review → test → fix → merge → close).
   d. After all sub-issues for a milestone are merged, close the milestone tracker.
   e. Update #93 roadmap and report progress to the user.
3. After all milestones are done, update `MEMORY.md` and post a final summary.

**Auto-resume after interruption**: Use the `/loop` skill with a self-paced
interval.  At each wakeup, check GitHub for open milestone sub-issues, pick
the next one, and resume work.  Never wait for the user to restart.

**Progress reporting**: Post one short message per completed milestone:
which PRs merged, issue numbers closed, and the updated roadmap link.

---

## Milestone Workflow

Milestones (#285, #286, …) are **tracking issues**, not implementation issues.
Each milestone contains several scoped sub-tasks.  The mandatory workflow is:

```
Milestone tracking issue
  └─ Sub-issue A  →  branch fix/issue-A-slug  →  PR  →  merge  →  close A
  └─ Sub-issue B  →  branch fix/issue-B-slug  →  PR  →  merge  →  close B
  └─ …
  └─ When ALL sub-issues are closed → close the milestone tracking issue
```

### Rules (apply to every milestone, do not repeat in user instructions)

1. **One issue → one PR**.  Never batch unrelated sub-tasks into a single PR.
   Split them even if they touch the same files.
2. **Open sub-issues first**, reference them from the milestone tracking issue
   body (update the checkbox list), then implement.
3. **Each PR follows the full review/test/issue-fix loop** (see above):
   self-review → cargo test → open issue(s) for any found problems → fix in
   same branch → post `gh pr review --comment` → merge.
4. **Close sub-issues immediately after their PR merges** — either via
   `closes #N` in the commit message (auto-close) or `gh issue close N`
   manually if the auto-close did not fire.
5. **Close the milestone tracking issue** once every sub-issue checkbox is
   ticked.  Update the Status Snapshot in #93 at that point.
6. **Branch naming for milestone sub-issues**:
   `feat/milestone-<letter>-<slug>`  (e.g. `feat/milestone-i-x86-fp`)
7. **Parallel agents** — independent sub-issues may be implemented by
   separate agents running in parallel worktrees (`isolation: "worktree"`).
   Sub-issues that depend on shared infrastructure (e.g., a new register
   class abstraction) must be sequenced: infrastructure PR first, then
   dependents.

---

## Code Quality Standards

Every PR merged into `main` must satisfy:

1. **`cargo test` all green** — no skipped tests, no `#[ignore]` added.
2. **Targeted tests** — every bug fix adds at least one regression test named
   after what it verifies (e.g. `udiv_uses_div_r_not_idiv_r`).
3. **Formatting + lints clean** — `cargo +stable fmt --all -- --check` and
   `cargo +stable clippy --locked --all-targets -- -D warnings` both pass.
   These are enforced by the `Quality Gates` workflow
   (`.github/workflows/quality-gates.yml`).
4. **Minimal diff** — only the lines necessary to fix the bug or implement
   the feature; no reformatting or unrelated cleanup.  (Exception: the
   one-time gate-establishing fmt/clippy PRs that brought the tree into
   compliance.)
5. **Squash merge** — one commit per PR on `main`; branch history preserved
   in the PR.
6. **Closes #N in commit message** — so GitHub auto-closes the issue.

---

## Phase Roadmap

| Phase | Crates | Status |
|-------|--------|--------|
| 1 — IR Foundation | `llvm-ir`, `llvm-ir-parser` | ✅ Complete |
| 2 — Analysis | `llvm-analysis` | ✅ Complete |
| 3 — Optimizations | `llvm-transforms` | ✅ Complete |
| 4 — x86_64 Backend | `llvm-codegen`, `llvm-target-x86` | ✅ Complete + reviewed |
| 5 — AArch64 + Bitcode | `llvm-target-arm`, `llvm-bitcode` | 🔲 Next |

For Phase 5 details see the open issue #7 and `CLAUDE.md` §"Phase 5".

---

## Memory & Context

Persistent cross-session notes live at:
```
/Users/yudong/.claude/projects/-Users-yudong-work-claude-LLVM-in-Rust/memory/MEMORY.md
```

**Always read `MEMORY.md` at the start of a session** to avoid re-doing work.
**Always update `MEMORY.md` after a phase completes** with new status, key
file paths, and design decisions.

Topic files in the same directory (`debugging.md`, `patterns.md`, etc.) hold
deeper notes; link to them from `MEMORY.md`.

---

## Commit Message Format

```
<imperative subject line, ≤72 chars> (closes #N)

<optional body: root cause, approach, notable decisions>
<blank line if body present>
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

PR body template:
```markdown
## Summary
- <bullet>

## Root cause / Design rationale
<paragraph>

## Test plan
- [ ] <new test name> — <what it verifies>
- [ ] All <X> existing tests pass

Closes #N

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

---
> Source: [yudongusa/LLVM-in-Rust](https://github.com/yudongusa/LLVM-in-Rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
