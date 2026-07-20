## rational-constitution

> Collaboratively evolve a minimal, rational constitution and related Acts using Markdown.

# AGENTS.md — Rational Constitution Project

## Mission
Collaboratively evolve a minimal, rational constitution and related Acts using Markdown.
All deliberation is recorded through automatically maintained ADR drafts; all final changes are validated, logged, and committed only with explicit consent.

---

## Guardrails
- Only edit files when asked; **/check-in** is the explicit commit/push trigger.
- During **/check-in**, commit all intended changes (constitution/acts/ADRs/tooling) plus the prepared `log/changelog.md` entry; ensure no unintended files are included.
- Keep constitutional/act text clean; no inline provenance markers or footers.
- Prefer small, auditable diffs; reference ADRs in ADRs, commits, and changelog (not inline in articles).

## Provenance (per file)
- No inline markers in governed texts (e.g., `constitution.md`, `acts/*.md`).
- Provenance is maintained via:
  - ADRs (drafts and finals) with clear “Changed Sections”,
  - commit messages referencing ADR IDs/titles, and
  - `log/changelog.md` entries linking ADRs to specific files/sections.
- A concise, out-of-band provenance index is kept at `log/provenance.md` mapping files → ADRs/commits. It is updated at `/record` and is safe to publish; it does not modify governed texts.
  
---

## Automatic ADR Handling

When conversation touches on constitutional or structural topics:

1. **Detect or create an active ADR draft** in `log/ideas/_drafts/` for the topic.
   - Name: `session-YYYYMMDD-HHMM-topic.md`
   - Include timestamp, summary, and incremental updates as discussion progresses.
2. **Append updates automatically** after each significant turn:
   - Add new arguments, tradeoffs, pros/cons, open questions.
   - Update Core alignment flags.
   - Do not overwrite previous text except to fix clarity/typos.
3. If conversation shifts topics:
   - Pause current ADR (record timestamp).
   - Start or switch to a new ADR draft.
4. ADR drafts remain until `/record` finalizes them into numbered ADRs.
5. Upon finalization via `/record`, drafts are promoted to numbered ADRs and the draft files are removed (not archived) by default. Git history preserves all draft evolution. If you explicitly mark a draft as Paused, it may remain in `_drafts/` until resumed or discarded.

Before `/check-in`, all active ADR drafts must be finalized via `/record`. The only drafts allowed to remain are those explicitly marked Paused by the user.

---

### Brainstorm Exception to Auto-Recording
- When Brainstorm mode is active, the assistant does not write to ADRs or files unless explicitly instructed.
- Instead, it maintains an internal/session scratchpad of discussion (concepts, alternatives, tradeoffs, open questions).
- On exit from Brainstorm, the assistant should first propose a structured summary of the discussion for appending to the active ADR draft; it will append only if you explicitly confirm. Normal auto-recording may resume thereafter.

This preserves the “always records” principle via deliberate, batched recording rather than per-turn writes.

---

## ADR Finalization Guidelines (Single Source of Truth)

When finalizing an ADR via `/record`, use promote-by-rename as the default:

- Move + rename the active draft from `log/ideas/_drafts/session-...-topic.md` to `log/ideas/adr-XXXX-kebab-title.md` (next number).
- Then make minimal finalize edits only:
  - Update Title/Number, `Status`, and `Date` (past tense voice).
  - Ensure sections match the ADR template (Context, Decision, Rationale, Alternatives, Consequences, Changed Sections, Links).
  - Add a concise “Changed Sections” block if governed texts are edited.
  - Add Links to related ADRs and relevant files.
- Remove the now-finalized draft from `_drafts/` (the git history preserves all iterations).

Notes:
- Prefer this flow over regenerating the ADR to avoid content drift and reduce risk/cost.
- Command docs should reference this section rather than duplicate guidelines.

---

## Authoring Aids (Pointers)

- Single source of truth for ADR authoring/finalization is this file (see “ADR Finalization Guidelines” above).
- Default finalization flow: promote-by-rename the active draft to `log/ideas/adr-XXXX-title.md`, then minimally update metadata and required sections.
- Provenance policy: governed texts remain clean (no inline markers). Maintain provenance via ADRs with “Changed Sections,” commit messages referencing ADR IDs, `log/changelog.md`, and `log/provenance.md`.
- Avoid scattering guidance across extra files or folders; link back here from command docs.

---

## Constitutional Core (Decision Filter)

All proposals must satisfy:

1. **Assume human nature** – individuals/corporations act in self-interest; design incentives accordingly.  
2. **Harness, don’t hope** – channel ambition toward public good; avoid dark incentives.  
3. **Power diffused, checked** – minimize concentrated authority; ensure retroactive accountability.  
4. **Radical transparency** – data, models, and actions open by default.  
5. **Democracy on outcomes** – citizens vote for goals, technocrats execute; measure all results.  
6. **Market where possible** – use externality pricing and performance competition.  
7. **Measurement over assertion** – every mandate/policy must define metrics and audit windows.

When proposing or revising anything, the AI must evaluate compliance with each tenet and record that evaluation in the ADR.

---

## Workflow Overview

### 1. Discussion & Auto-Recording
- As ideas evolve, Cursor continuously appends updates to the active ADR draft.
- Each turn may add:
  - New arguments
  - Pros/Cons
  - Core compliance notes
  - Open questions (to `log/open-questions.md` if appropriate)

### 2. Convergence
- When prompted via `/converge`, summarize and assess readiness:
  - Identify alignment/conflicts with Core.
  - Recommend: Reject, Needs Work, or Ready.

### 3. Recording & Implementation
- `/record` finalizes the ADR:
  - Numbered ADR created from draft.
  - Constitution/Acts edited if “Ready”.
  - Changelog updated automatically.

### 4. Validation & Commit
- `/validate` checks consistency, Core adherence, and completeness.
- `/check-in` adds changelog entry, prepares commit message, and pushes only after confirmation.

### Check-in Preconditions
- All intended edits (ADR finalization, constitution/acts changes) must already be applied and reviewed.
- `/check-in` will refuse to run if non-changelog edits are pending (dirty working tree with unstaged/unsaved changes).
- The only write performed by `/check-in` is inserting the prepared changelog entry.

---

## File Conventions

| Type | Location | Notes |
|------|-----------|-------|
| Constitution | `constitution.md` | Canonical master text |
| Acts | `acts/*.md` | Supplemental laws |
| ADR Drafts | `log/ideas/_drafts/` | Auto-updating working discussions |
| ADR Final | `log/ideas/adr-XXXX-title.md` | Immutable decision records |
| Changelog | `log/changelog.md` | Auto-updated at /record |
| Open Questions | `log/open-questions.md` | Append-only list, linked from ADRs |

---

## Editing Rules

- Preserve structure, numbering, and voice.
- Keep the constitutional and act texts **clean** (no inline ADR references in articles/sections).
- Record provenance **once per file** using the Provenance Policy below.
- For multi-section edits, keep diffs minimal and atomic when possible.
- Cross-reference ADRs in the **changelog** and within the **ADR** itself (pointing to the exact sections changed).
  
---

## Provenance Policy

**Purpose:** maintain clean text while preserving full traceability.

- No inline provenance markers or footers in governed texts (e.g., `constitution.md`, `acts/*.md`).
- Provenance is maintained via:
  - ADRs (drafts and finals) with clear “Changed Sections,”
  - commit messages referencing ADR IDs/titles, and
  - `log/changelog.md` entries linking ADRs to specific files/sections.
- A concise, out-of-band provenance index at `log/provenance.md` maps files → ADRs/commits and sections; it is updated at `/record` and does not modify governed texts. It is safe to publish and can be regenerated from ADRs and git history.

---

## Summary of Commands

| Command | Purpose |
|----------|----------|
| `/converge` | Summarize and evaluate discussion state |
| `/record` | Finalize ADR and apply edits if Ready |
| `/validate` | Pre-commit verification |
| `/check-in` | Log, commit, and push on confirmation |
| (see `.cursor/commands/`) | Project-specific commands live under `.cursor/commands/` |

---

## Example Flow

1. **Discuss freely.** Cursor auto-records into current ADR draft.
2. **Use `/converge`** to review and gauge readiness.
3. **Decide:** if Ready → `/record`; if not approving now, either `/record` as Rejected or explicitly mark the draft as Paused.
4. **Run `/validate`** to verify compliance and coherence.
5. **Approve `/check-in`** to finalize, commit, and push.

---

## Conversation Modes

### Brainstorm Mode
- Purpose: Explore, hypothesize, stress-test, and refine ideas without making file changes.
- Assistant behavior:
  - No file edits and no ADR updates unless explicitly asked.
  - Socratic probing: surface assumptions, alternatives, invariants, edge cases.
  - Provide options with tradeoffs and “attack” tests before any recommendation.
  - Draft candidate text inline only if requested; do not apply.
- Exit:
  - User verbally indicates (“Execute: ON” or similar), the assistant proposes a structured ADR summary and appends it only upon explicit confirmation, then normal operations may resume.

Note: `/record` remains the sole action that promotes a draft to a numbered ADR and updates the changelog.

---

### Key Principles

- The AI **always records**, **never forgets**, and **never commits without you**.  
- Every decision traces back to an ADR.  
- Every ADR traces back to transparent reasoning.  
- Every change traces back to both.

This guarantees a complete, auditable record of constitutional evolution.

---

---
> Source: [copperdogma/rational-constitution](https://github.com/copperdogma/rational-constitution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
