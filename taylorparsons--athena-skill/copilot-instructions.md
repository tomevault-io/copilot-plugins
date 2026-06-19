## athena-skill

> Turn a PRD into shipped changes using a strict, repeatable ATHENA loop that captures customer requests in docs/requests.md, decisions in docs/decisions.md, and per-feature specs in docs/specs/FEATURE_ID/spec.md (with Sources/Verifies/Implements traceability), then reads docs/PRD.md and docs/progress.txt to execute one small task at a time with validation and session notes. Use when the user wants iterative PRD-driven delivery with explicit task tracking and an audit trail from raw requests to final requirements.


# ATHENA Loop

Follow this loop every session.

Ground rules:
- Treat `docs/requests.md` as an append-only log of raw customer requests (inputs).
- Treat `docs/decisions.md` as an append-only decision trail that explains how inputs became PRD requirements.
- Treat `docs/specs/<feature-id>/spec.md` as the human-readable feature spec with traceability tags (Sources/Verifies).
- Treat `docs/specs/<feature-id>/tasks.md` as the task list with traceability tags (Implements).
- Treat `docs/TRACEABILITY.md` as the entry point explaining how to follow the audit trail.
- Treat `docs/PRD.md` and `docs/progress.txt` as the source of truth for execution.
- Do not make any PRD/code changes until the current session’s customer input has been captured as a new `CR-...` entry (verbatim) in `docs/requests.md`.
- If the project is a Git repo, create local commits as tasks are completed with traceability pointers back to `docs/` (see “6.5) Create a Traceable Local Commit”). Never push to a remote unless the user explicitly requests it.
- If a feature involves UI/UX work, offer the `daisy` skill as an option/framework for the task.

## 0) Capture the Customer Request (Input)

At the start of each session, record the current customer request **verbatim**:
- Ensure `docs/requests.md` exists (create it using the template below if missing).
- Append a new entry with a unique ID and timestamp.
- If the request includes secrets (API keys, passwords, tokens), redact them before writing.

If you do not have an explicit customer request for the current session:
- Ask the user for it before changing `docs/PRD.md`.

## 0.25) Capture the Agent / Skill Context (for context resets)

ATHENA does not automatically track which skills/agents were used. To make sessions recoverable after a context reset, explicitly record the execution context in `docs/progress.txt`.

At the start of each session, set these header fields in `docs/progress.txt`:
- `Agent:` (e.g., `Codex CLI`)
- `Model:` (if known)
- `Skills:` (e.g., `athena`, plus any other skills you actually used during the session)

## Task ↔ Skill Mapping Convention

To preserve a clear mapping between the task you are working on and the skill(s) you used, annotate the task lines in `docs/progress.txt` with a `Skills:` tag.

Format (recommended):

```text
- <task line...> (Skills: athena, <other-skill>, ...)
```

Rules:
- Every task listed under `IN PROGRESS` MUST include `Skills: ...`.
- If you used additional skills mid-task, update that task’s `Skills:` list before ending the session.

## 0.5) Capture Decisions When You Interpret or Change Scope

Whenever you:
- Interpret ambiguous wording,
- Choose between options,
- Decide tradeoffs,
- Change scope,
- Update `docs/PRD.md` in a way that is not a verbatim copy of the request,

Append a decision entry to `docs/decisions.md`:
- Use a unique ID (`D-YYYYMMDD-HHMM`).
- Reference the relevant request IDs (`CR-...`).
- Reference the PRD section(s) impacted (e.g., heading name).
- Keep it short, factual, and testable.

## 1) Ensure Source-of-Truth Files Exist

Check for these files first:
- `docs/requests.md`
- `docs/decisions.md`
- `docs/TRACEABILITY.md`
- `docs/PRD.md`
- `docs/progress.txt`

If any file is missing, create it before doing anything else.

When creating missing files:
- Keep `docs/PRD.md` minimal and factual. Do not invent requirements.
- Initialize `docs/progress.txt` using the template in the “Progress Log Template” section.
- Initialize `docs/requests.md` using the template in the “Customer Request Log Template” section.
- Initialize `docs/decisions.md` using the template in the “Decision Log Template” section.
- Initialize `docs/TRACEABILITY.md` using the template in the “Traceability Entry Point Template” section.
- If the project is already a Git repo with existing history and these audit files were missing, generate a historical audit trail from Git (see “1.25) Bootstrap Historical Audit from Git History”).

## 1.25) Bootstrap Historical Audit from Git History (for adopted repos)

If you are adding ATHENA to an existing project that:
- has Git enabled (a `.git/` directory), and
- is not already using the ATHENA audit files,

create a **historical audit trail derived from Git history** so there is an explicit “before ATHENA” record.

Rules:
- Do **not** fabricate customer requests. Git history is not “verbatim customer input”.
- Keep customer inputs in `docs/requests.md` as verbatim going forward (starting with the current session’s request).
- Store derived history in `docs/audit/git-history.md` and clearly label it as derived.

Preferred implementation:
- Run the bundled script: `python3 <CODEX_HOME>/skills/athena/scripts/bootstrap_git_audit.py --out docs/audit/git-history.md`
- Then update `docs/TRACEABILITY.md` to include a pointer to `docs/audit/git-history.md`.

## 2) Read First, Then Act

Check auto-memory first (loaded by Claude Code before this session started). If `Athena Active Context` memory is present:
- Active feature, goal, and task state are already known — skip reading `docs/athena-index.md` and `docs/progress.txt`
- Load only: `docs/requests.md`, `docs/decisions.md`, `docs/PRD.md`, and `docs/specs/<active-feature-id>/`

If memory is absent (no `Athena Active Context` entry):
- Load the full stack: `docs/athena-index.md` → `docs/requests.md` → `docs/decisions.md` → `docs/TRACEABILITY.md` → `docs/PRD.md` → `docs/progress.txt` → active spec(s)

**Token optimization**: The memory file is written by Owl at session start and contains the active feature ID, goal, IN PROGRESS/NEXT tasks, and active features list. When present, it replaces the need to parse `athena-index.md` and `progress.txt` at session open.

## Owl of Athena — Archive Management

Owl is a Claude Code sub-agent (`.claude/agents/owl-of-athena.md`) that handles all archive and housekeeping operations. Use the Agent tool to dispatch to it — do not load archived specs directly.

**Dispatch to Owl in these situations:**

| Situation | Dispatch prompt |
|---|---|
| User asks about an archived feature | `"Retrieve summary for <feature-id>"` |
| User searches history | `"Search archived features for '<keyword>'"` |
| Feature fully done (tasks.md + spec.md + PRD) | `"Archive feature <feature-id>"` |
| athena-index.md appears stale | `"Run update-index"` |

**A feature is fully done when ALL three are true:**
1. `tasks.md` — no items under NEXT or IN PROGRESS
2. `spec.md` — `Status: Done`
3. `PRD.md` — feature appears as shipped/complete

When all three pass, dispatch `"Archive feature <feature-id>"` to Owl. The SessionStart hook runs `prune-done` at session open to remove closed feature history from `progress.txt`.

Summarize the current goal in one sentence before making changes.

If the latest request in `docs/requests.md` is not reflected in `docs/PRD.md`:
- Update `docs/PRD.md` first (cite the request ID in the PRD).
- Then continue the loop.

If you must interpret the request to update the PRD:
- Write a decision in `docs/decisions.md` (cite both `CR-...` and the affected PRD section).
- Then update `docs/PRD.md` and cite the decision ID(s) next to the requirement(s).

## PRD Traceability Convention

When adding or changing any requirement in `docs/PRD.md`, append a `Sources:` tag to the same line (or immediately below it) so every requirement is traceable:

```text
<requirement text> (Sources: CR-YYYYMMDD-HHMM; D-YYYYMMDD-HHMM)
```

Rules:
- Always include at least one `CR-...`.
- Include `D-...` whenever interpretation/tradeoffs were needed.
- If multiple sources apply, list them comma-separated: `Sources: CR-..., CR-...; D-..., D-...`.

## 2.5) Maintain a Feature Spec + Task List (Audit Trail)

For the current request, establish a `FEATURE_ID` (ask the user if unclear).

Recommended formats:
- `NNN-short-name` (preferred if your repo already uses numbering)
- `YYYYMMDD-short-name` (safe default in any repo)

Then ensure these exist (create if missing using templates below):
- `docs/specs/<FEATURE_ID>/spec.md`
- `docs/specs/<FEATURE_ID>/tasks.md`

Rules for writing/updating the feature spec (`docs/specs/<FEATURE_ID>/spec.md`):
- Keep it human-readable: user stories + Given/When/Then acceptance scenarios.
- Every functional requirement MUST have a stable ID (`FR-001`, `FR-002`, …).
- Every requirement MUST include `Sources:` with at least one `CR-...`, and `D-...` when interpretation/tradeoffs were required.
- Every acceptance scenario MUST include `Verifies: FR-...`.

Rules for writing/updating the task list (`docs/specs/<FEATURE_ID>/tasks.md`):
- Every task MUST have an ID (`T-001`, `T-002`, …).
- Every task MUST include `Implements: FR-...`.

Before writing any code, enforce traceability:
- If you add/change a requirement, update `Sources:` and add/update acceptance scenarios.
- If you add/change tasks, ensure each task references existing `FR-...` IDs.
- If any link is missing or invalid, fix the documents first.

## 3) Select the Next Task

Identify the next unfinished task in `docs/progress.txt`.

If no tasks exist:
- Derive a short, concrete task list from `docs/PRD.md`.
- Write those tasks into `docs/progress.txt` under `NEXT`.
- Then select the first task.

If the PRD is unclear:
- Do not guess.
- Add a brief question under “Risks / open questions” in `docs/progress.txt`.
- Choose the safest, smallest unambiguous task, or stop and ask the user.

When you move a task into `IN PROGRESS`:
- Add `(Skills: ...)` to that task line (minimum: `athena`).

## 4) Implement One Small Task

Implement only one task at a time with small, reviewable changes.

Work style constraints:
- Prefer the smallest change that satisfies the PRD.
- Do not invent requirements.
- Keep changes scoped to the current task.
- Avoid refactors unless required by the current task.
- Do not delete files unless explicitly required by the task. If deletion is necessary, record it in `docs/progress.txt`.

## 5) Run Relevant Checks

Run relevant validation based on the repository:
- Tests
- Lint
- Typecheck
- Build

If checks cannot be run, record why in `docs/progress.txt`.

## 6) Update the Progress Log at the End

At the end of the session, update `docs/progress.txt`:
- Mark completed tasks as `DONE` with date/time.
- Add newly discovered follow-up tasks under `NEXT`.
- Note commands run and their outcomes.
- Note skills used (update the `Skills:` header if it changed during the session).
- Record decisions, tradeoffs, and open questions.

Always keep `IN PROGRESS` to a single task.

After updating `docs/progress.txt`, run `./scripts/owl write-memory` to refresh the memory file with current task state for the next session.

## 6.25) Canonical Merge / Check-in Reconciliation (Required)

Use this single checklist before finalizing a task and again after any merge to `main` (local or remote) so PRD/spec/tasks/progress remain synchronized.

Checklist:
1) Spec status update
- If code/tests now satisfy feature FRs, set `Status: Done` in `docs/specs/<FEATURE_ID>/spec.md`.

2) PRD backlog reconciliation
- Update `docs/PRD.md` “Next / backlog” to remove, move, or mark shipped items.
- Include the CR ID on shipped items when applicable.
- Include at least one concrete evidence path per shipped item (for example `app/static/js/ui.js`).

3) Progress log sync
- Ensure `docs/progress.txt` marks completed tasks as `DONE` with timestamp.
- If PRD/spec status and DONE state diverge, reconcile immediately.

4) Merge evidence
- If merge has happened, record merge commit hash (or merge PR ID) in `docs/progress.txt` under `NOTES`.
- If merge has not happened yet, record `pending merge` in `docs/progress.txt` under `NOTES` and do not mark PRD items as shipped.

5) Exception handling
- If any reconciliation step cannot be completed, record the reason in `docs/progress.txt` under `NOTES`.

## 6.5) Create a Traceable Local Commit (Git repos only)

Goal: enhance the audit trail for long-running work and provide a local backup.

If `.git/` exists:
- Stage and commit changes at least when a task moves to `DONE` (and optionally after each meaningful checkpoint).
- The commit message MUST include traceability pointers back to the ATHENA artifacts (CR/D/FEATURE_ID/T-... and docs paths).
- Record the commit hash in `docs/progress.txt` under `NOTES` (or next to the task) so the audit trail links to Git history.

Hard rules:
- Never run `git push` (or open/submit PRs) unless the user explicitly asks.
- Do not commit secrets. If secrets are discovered, remove them and note the incident in `docs/progress.txt`.

Recommended workflow:
- Stage (default): broad stage with guardrails via helper prechecks (blocks `.DS_Store`, common temp/cache paths, likely secret patterns, and large artifacts).
- Stage (explicit): pass `--paths <path1> <path2> ...` for custom scope.
- Stage (docs-only): pass `--docs-only` to use ATHENA traceability docs path-scoped staging (`spec/tasks/progress/PRD/TRACEABILITY/requests/decisions`).
- Stage (override): pass `--skip-staging-precheck` only when blocked files are intentionally included.
- Review: `git diff --cached`
- Commit using the helper script:
  - `python3 <CODEX_HOME>/skills/athena/scripts/commit_with_traceability.py --feature <FEATURE_ID> --task <T-001> --summary "<what changed>" --cr <CR-...> --decisions <D-...,...> [--paths ... | --docs-only | --all-changes]`

Commit message format (recommended):
- Subject: `T-001: <summary> (Feature: <FEATURE_ID>)`
- Body:
  - `Input: CR-...`
  - `Decisions: D-...`
  - `Spec: docs/specs/<FEATURE_ID>/spec.md`
  - `Tasks: docs/specs/<FEATURE_ID>/tasks.md`
  - `Progress: docs/progress.txt`

## Context Restore (After a Context Reset)

When context is lost, regenerate a “resume prompt” from repo state and paste it into the new chat so the relevant skills re-trigger.

Preferred:
- Run: `python3 <CODEX_HOME>/skills/athena/scripts/validate_progress_log.py --repo .`
- Run: `python3 <CODEX_HOME>/skills/athena/scripts/print_resume_prompt.py --repo .`
- Paste the printed prompt into Codex.

## Plan Interaction (Use create-plan Skill)

When the user explicitly asks for a plan:
- Use the `create-plan` skill.
- Keep planning read-only.
- After planning, resume the ATHENA loop for execution.

## Customer Request Log Template (`docs/requests.md`)

Use this structure (append-only):

```text
# Customer Requests (append-only)

## CR-YYYYMMDD-HHMM
Date: YYYY-MM-DD HH:MM
Source: <chat/email/ticket/etc>

Request (verbatim):
<paste request here>

Notes:
- <optional clarifications>
```

## Traceability Entry Point Template (`docs/TRACEABILITY.md`)

Use this structure:

```text
# Traceability (How to follow the audit trail)

Start here:
0) If this repo adopted ATHENA after it already had history, review `docs/audit/git-history.md` (derived from Git; not customer verbatim).
1) Find the relevant raw request in `docs/requests.md` (CR-...).
2) Read linked interpretations/tradeoffs in `docs/decisions.md` (D-...).
3) Open the feature spec at `docs/specs/<FEATURE_ID>/spec.md`.
   - Requirements use IDs (FR-...) and include `Sources: CR-...; D-...`.
   - Acceptance scenarios include `Verifies: FR-...`.
4) Open the feature task list at `docs/specs/<FEATURE_ID>/tasks.md`.
   - Tasks include `Implements: FR-...`.
5) Review execution notes in `docs/progress.txt` for commands, outcomes, and completion.
```

## Decision Log Template (`docs/decisions.md`)

Use this structure (append-only):

```text
# Decisions (append-only)

## D-YYYYMMDD-HHMM
Date: YYYY-MM-DD HH:MM
Inputs: CR-YYYYMMDD-HHMM[, CR-...]
PRD: <section/heading(s) impacted>

Decision:
<what you decided>

Rationale:
<why this is the safest/most correct choice>

Alternatives considered:
- <option> (rejected because <reason>)

Acceptance / test:
- <how to verify this decision/requirement is satisfied>
```

## Feature Spec Template (`docs/specs/<FEATURE_ID>/spec.md`)

Use this structure:

```text
# Feature Spec: <FEATURE_ID>

Status: Draft | Active | Done
Created: YYYY-MM-DD HH:MM
Inputs: CR-YYYYMMDD-HHMM[, CR-...]
Decisions: D-YYYYMMDD-HHMM[, D-...]

## Summary
- <one paragraph of what/why>

## User Stories & Acceptance

### US1: <title> (Priority: P1)
Narrative:
- As a <user>, I want <capability>, so that <benefit>.

Acceptance scenarios:
1. Given <state>, When <action>, Then <outcome>. (Verifies: FR-001, FR-002)

## Requirements

Functional requirements:
- FR-001: <requirement text>. (Sources: CR-YYYYMMDD-HHMM; D-YYYYMMDD-HHMM)
- FR-002: <requirement text>. (Sources: CR-YYYYMMDD-HHMM)

Non-functional requirements (optional):
- NFR-001: <requirement text>. (Sources: CR-...; D-...)

## Edge cases
- <case> (Verifies: FR-...)
```

## Feature Task List Template (`docs/specs/<FEATURE_ID>/tasks.md`)

Use this structure:

```text
# Tasks: <FEATURE_ID>

Spec: docs/specs/<FEATURE_ID>/spec.md

## NEXT
- T-001: <task>. (Implements: FR-001)
- T-002: <task>. (Implements: FR-002)

## IN PROGRESS
- <at most one task>

## DONE
- [YYYY-MM-DD HH:MM] T-000: <task>. (Implements: FR-...)
```

## Progress Log Template (`docs/progress.txt`)

Use this structure:

```text
Session: YYYY-MM-DD HH:MM
Agent: Codex CLI
Model: <if known>
Skills: athena[, ...]
Feature: <FEATURE_ID>
Input: CR-YYYYMMDD-HHMM
Decisions: D-YYYYMMDD-HHMM[, D-...]
Spec: docs/specs/<FEATURE_ID>/spec.md
Tasks: docs/specs/<FEATURE_ID>/tasks.md
Git history (optional): docs/audit/git-history.md
Goal: <one sentence>

DONE
- [YYYY-MM-DD HH:MM] <task>. (Skills: athena[, ...])

IN PROGRESS
- <task>. (Skills: athena[, ...])

NEXT
- <task>
- <task>

NOTES
- Commands run: <cmd> -> <result>
- Decisions: <what and why>
- Risks / open questions: <items>
```

---
> Source: [taylorparsons/athena-skill](https://github.com/taylorparsons/athena-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
