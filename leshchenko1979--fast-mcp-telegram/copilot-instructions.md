## fast-mcp-telegram

> Required plan-first workflow for any implementation request (steps 1–5)


# Implementation workflow

Any code change follows **steps 1–5**. The user approves at **step 3 only** (the plan), not implementation diffs.

**Start:** Plan mode before step 1. No code until explicit step 3 approval.

## Steps

| Step | Mode | Action | Gate |
| ---- | ---- | ------ | ---- |
| **1** | Plan | Scope, files, exit commands, clean-break checks; link checklist rows; **learning** fields (below). Plan file: `~/.cursor/plans/<task>_*.md` only (not in-repo). If scope is too big → [scope split](#scope-split-steps-12). | Hypothesis, success signal, and kill criteria recorded |
| **2** | Plan | Pre-impl review (`code-reviewer`, readonly): trim scope; favor simplicity; DX if user-facing; **Deferred** for cuts; or **phase split** (below). Revise plan; **no code**. | Scope trimmed **or** split agreed; plan favors simplicity; exits set; **Deferred** if anything cut; plan stand-alone; incremental edits keep **previous plan** |
| **3** | Plan → Agent | Present plan; wait for approval; roadmap ask if **Deferred** (below). | Roadmap ask if **Deferred**; explicit approval |
| **4** | Agent | Run exit tests/commands; fix failures. No extra confirmation prompts. | Exits green |
| **5** | Agent | **Closeout** (below): one pass — review, fixes, docs. | Closeout checklist done |

Before step 4: reread [CONTRIBUTING.md](CONTRIBUTING.md) (Design Philosophy, Code Quality, Development Workflow). Boundaries: design philosophy and tool-count guidelines in CONTRIBUTING; session/MCP patterns in [systemPatterns.md](.cursor/memory-bank/systemPatterns.md).

## Step 1 — Learning fields

Record in the plan (short bullets; full sentences optional):

| Field | Purpose |
| ----- | ------- |
| **Hypothesis** | What we believe this change will prove or enable |
| **Success signal** | What “worked” looks like beyond exit commands (user outcome, metric, or spike claim) |
| **Kill / stop** | When to abandon or narrow scope before more implementation (failed exit class, wrong approach, scope creep) |

Exit commands remain the **technical** success bar; these fields tie the task to build-measure-learn.

## Step 2 — Plan review

**Goals:** Smaller scope; simplicity (design, APIs, diffs); better user-facing DX — without telegram-style compression.

| Lens | When relevant |
| ---- | ------------- |
| **Time-to-results** | Fewer commands to first green; mock/no-key path in exits |
| **Ceremony vs value** | Extend existing tools/params; drop boilerplate that does not buy regression value |
| **Errors & discoverability** | Failures name the fix (path, flag, field); docs/README in-plan when behavior changes |
| **Spikes** | Comparative or API claims → [feature-development skill](.cursor/skills/feature-development/SKILL.md) research phase; split orchestration vs domain |

DX vs minimal scope / simplicity → **record tradeoff** in plan (default: smaller scope and simpler shape unless user asked for DX).

**Deferred:** One line per cut (what + why). Never silently drop user- or spike-requested work.

## Scope split (steps 1–2)

When scope is still too large after trimming (or the user asked for a multi-part deliverable), **offer a consecutive phase split** instead of a single oversized plan. Do not implement until one phase is approved.

**Offer two paths** (user picks; default **A** if the full plan is not yet written):

| Path | When | Next |
| ---- | ---- | ---- |
| **A — Fresh phase** | No stand-alone plan yet, or phases need different exits/files | Document split in roadmap (below) → **step 1** for **phase 1 only** |
| **B — Slice plan** | A stand-alone plan already exists with clear boundaries | Document split in roadmap → **slice** plan to phase 1 (+ **Phases 2+** section with scope, files, exits per later phase) → **step 3** for phase 1 only |

**Phase split contents** (in plan and roadmap):

- **Phase 1 … N** titles and one-line goal each
- Per phase: scope, files, exit commands, checklist row links
- **Out of scope** for phase 1 (explicitly parked in later phases)
- **Deferred** only for cuts *within* a phase, not for whole later phases

**Roadmap (before step 3 on path A, or with the slice on path B):** Record the split where the user agrees (same targets as [Deferred → roadmap](#step-3--approval)) — typically [docs/Roadmap.md](docs/Roadmap.md), memory bank `activeContext.md`, or a short research note. Do not edit roadmap until the user confirms target; then update in-session. Later phases: after phase *k* **closeout** (step 5), **step 1** for phase *k+1* (or slice from the master plan if path B).

**Approval:** Step 3 covers **current phase only**. Phases 2+ need their own **2 → 3** when started.

**Plan quality (step 3):** Full sentences; explicit scope, files, exits, rationale; learning fields from step 1. After feedback: revise **incrementally** in the same file; keep **previous plan**; state what changed. Current plan must **stand alone**.

## Step 3 — Approval

Stop and wait for the user.

**Deferred → roadmap:** If **Deferred** is non-empty, ask where to park items **before** approval. Do not edit roadmap docs until the user answers; record choice in the plan (`Phase N` / roadmap item / research note / none). Update roadmap in-session only on yes.

| Target | Use for |
| ------ | ------- |
| [docs/Roadmap.md](docs/Roadmap.md) | Feature lanes, phases, follow-up work |
| [.cursor/memory-bank/activeContext.md](.cursor/memory-bank/activeContext.md) | Current focus and next steps |
| [.cursor/memory-bank/progress.md](.cursor/memory-bank/progress.md) | Functional status and known issues |
| [docs/research/](docs/research/) | Spikes and design notes |

| User response | Next |
| ------------- | ---- |
| Feedback / change request | → step 2: vs CONTRIBUTING, memory bank, codebase; accept, push back, or smaller alternative; record in plan or docs; re-run pre-impl review; preserve **previous plan**. No code. |
| Explicit approval | Reread CONTRIBUTING.md; Agent mode; step 4. |

Repeat **2 → 3** until explicit approval.

## Step 5 — Closeout

Single agent pass after step 4 is green. Use `code-reviewer` (readonly) unless the diff is trivial docs-only.

| # | Check |
| - | ----- |
| 1 | Diff matches approved plan; no unapproved scope |
| 2 | No legacy paths or dual APIs unless the plan explicitly allowed a migration window |
| 3 | Re-run exit commands if review or fixes changed behavior |
| 4 | Fix **blocking** findings only; do not expand scope |
| 5 | **Success signal** — note in plan or chat whether the step 1 hypothesis held |
| 6 | Update [CONTRIBUTING.md](CONTRIBUTING.md) or memory bank when the approved plan required durable doc changes |
| 7 | Memory bank `activeContext.md` / `progress.md` when the approved plan required status tracking |

## After close

Optionally remind: `/summarize` to compact context (user-initiated).

---
> Source: [leshchenko1979/fast-mcp-telegram](https://github.com/leshchenko1979/fast-mcp-telegram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-11 -->
