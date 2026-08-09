## lightroom-ai-workflow

> Build a Windows-first Lightroom Classic exposure assistant.

# AGENTS.md

## Project Mission

Build a Windows-first Lightroom Classic exposure assistant.

The user opens a Lightroom folder, selects the intended photos, and runs
AI Exposure Assist. The system renders Lightroom previews, asks a vision
model to judge exposure consistency, writes approved exposure changes to
XMP sidecars, and returns a result report.

The MVP adjusts exposure only.

## Required Read Order

Before changing any file, read these files in order:

1. `AGENTS.md`
2. `docs/ARCHITECTURE.md`
3. `docs/XMP_SAFETY.md`
4. `docs/AI_JUDGE_CONTRACT.md`
5. `work-orders/CURRENT_WORK_ORDER.md`
6. The active work order referenced by `CURRENT_WORK_ORDER.md`

Do not begin implementation until the active work order is identified.

## Sources of Authority

Authority order:

1. Active Work Order
2. `AGENTS.md`
3. `docs/XMP_SAFETY.md`
4. `docs/ARCHITECTURE.md`
5. Existing tests
6. Existing implementation
7. README

When authorities conflict, stop and report the conflict.

## MVP Workflow

The canonical workflow is:

1. User opens the intended Lightroom Classic folder.
2. User selects the intended photos.
3. Lightroom plug-in renders JPEG previews.
4. Plug-in writes an ordered manifest.
5. Python CLI validates the job.
6. Vision AI returns one decision per image.
7. Python validates and clamps decisions.
8. Existing XMP files are backed up.
9. Only `crs:Exposure2012` may be changed.
10. A result report is written.
11. User reads metadata back into Lightroom.
12. User reviews, rejects unwanted photos, and exports manually.

## Non-Negotiable Boundaries

- Do not edit RAW, NEF, JPEG originals, or Lightroom Catalog files.
- Do not read or write `.lrcat`, `.lrcat-wal`, `.lrcat-shm`, or `.lrdata`
  directly in the MVP.
- Do not modify EXIF camera-capture fields.
- Do not modify White Balance, Contrast, Highlights, Shadows, Crop,
  Masks, Keywords, Rating, Label, Sharpening, or Noise Reduction.
- The only editable Lightroom development property is
  `crs:Exposure2012`.
- Back up every affected XMP before any real write.
- Default execution mode is `dry_run`.
- Never delete or move user photographs.
- Reject decisions are suggestions only in the MVP.
- Do not automate final export in the MVP.
- Never store API keys or secrets in tracked files.

## XMP Rules

- Treat `crs:Exposure2012` as an EV value.
- New exposure equals existing exposure plus validated AI delta.
- Preserve all unrelated XML elements, attributes, namespaces,
  whitespace where practical, and file encoding.
- Write through a temporary file and replace atomically.
- A failed write must leave the original XMP intact.
- Missing, malformed, or ambiguous XMP must stop that image and create
  a review result. Do not guess.

## AI Decision Rules

- Produce exactly one decision for every manifest image.
- Never invent filenames or file paths.
- Preserve manifest order.
- `delta_ev` must be numeric.
- Clamp `delta_ev` to the configured maximum.
- Low-confidence decisions must not be applied automatically.
- AI output is untrusted input and must be schema validated.
- The AI never writes files directly.

## Seven Execution Rules

1. **Task Classification**  
   Classify the task, scope, risk level, and required evidence before editing.

2. **Define Done First**  
   Identify acceptance criteria and completion evidence before implementation.

3. **Parallel Evidence Gathering**  
   Inspect repository truth, relevant files, tests, Git state, and governing
   documents before choosing a solution.

4. **Single Recommendation**  
   Once sufficient evidence exists, choose one best bounded approach.
   Do not delegate routine technical decisions back to the user.

5. **Surgical Change**  
   Make the smallest correct change. Touch only files and behavior required
   by the active Work Order.

6. **Verify by Execution**  
   Prove behavior by running the required tests, commands, or validation.
   Reading code or relying on a worker report is not sufficient evidence.

7. **Outcome-First Reporting**  
   Report the implemented outcome, validation evidence, Git scope, and
   remaining risks. Avoid unnecessary process narration.

## Four Common AI Failure Modes

1. **Memory Over Repository Truth**  
   Repository files, Git status, current HEAD, and executed evidence always
   override memory, prior summaries, and stale reports.

2. **Treating Worker Reports as Final Evidence**  
   A worker report is a claim. The final reviewer must inspect the actual
   diff, validation output, and repository state.

3. **Leaving the Proof Chain Open**  
   Work is not complete until acceptance criteria, tests, diff review,
   allowed-file scope, and Git status have all been verified.

4. **Unauthorized Scope Expansion**  
   Do not add refactors, frameworks, features, dependencies, or cleanup
   outside the active Work Order, even when they appear beneficial.

## Engineering Rules

- Work on one bounded Work Order at a time.
- Make the smallest safe change satisfying the Work Order.
- Do not redesign architecture unless explicitly required.
- Do not add frameworks without a demonstrated need.
- Prefer the Python standard library when practical.
- Add or update tests for every behavior change.
- Do not use broad staging commands such as `git add .`.
- Do not commit secrets, runtime jobs, previews, logs, or XMP backups.
- Do not commit unless the active Work Order explicitly authorizes it.

## Required Preflight

Before editing, report internally:

- Active Work Order
- Allowed files
- Forbidden files
- Expected behavior change
- Required validation
- Current Git status
- Dry-run or real-write mode

Unexpected dirty files are a stop condition.

## Stop Conditions

Stop without broad cleanup when:

- No active Work Order exists.
- Work Order scope is ambiguous.
- Git contains unexpected changes.
- Lightroom SDK behavior required by the task is unverified.
- The implementation would need direct Catalog or Preview-cache access.
- An XMP file cannot be parsed safely.
- Backup creation fails.
- AI output does not match the required schema.
- Tests fail outside the bounded change.
- A destructive action would be required.
- Credentials or real API charges require owner authorization.

## Completion Gate

A task is complete only when:

- Acceptance criteria are satisfied.
- Relevant tests pass.
- Dry-run evidence is produced where applicable.
- No forbidden file was changed.
- Git diff contains only task files.
- Remaining risks are reported.
- The active Work Order is updated truthfully.

## Final Report Format

Report only:

- Work Order
- Files changed
- Behavior implemented
- Validation performed
- Test result
- Remaining risks or stop condition

Do not claim success without evidence.

## Documentation Authority and Index

`docs/INDEX.md` is the canonical documentation index for this repository.

Before implementation, the coding agent must use `docs/INDEX.md` to identify:

- Documents governing the active Work Order
- Architecture and safety contracts affected by the task
- Documents that may require updates before closeout
- Historical or superseded documents that must not be treated as current authority

A document not listed in `docs/INDEX.md` must not silently become a new
source of authority. Add it to the index when the active Work Order
authorizes creation of that document.

Documentation authority does not override the active Work Order.
The authority order defined in this file remains in effect.

## Documentation Closeout Gate

Every Work Order must leave the repository documentation consistent with
the implemented repository truth.

Before declaring a Work Order complete, the coding agent must perform a
documentation impact review.

The review must determine whether the task changed any of the following:

- Architecture or component boundaries
- Runtime or user workflow
- Configuration fields or defaults
- Commands, installation, or operating instructions
- Data formats, schemas, or API contracts
- Safety boundaries or forbidden behavior
- File or directory structure
- Dependencies or supported environments
- Known limitations, risks, or unresolved decisions
- Work Order status and current-work pointer

For every affected item, update the corresponding canonical document in
the same Work Order when the file is authorized.

At minimum, review:

1. `docs/INDEX.md`
2. `README.md`
3. Relevant documents listed in `docs/INDEX.md`
4. `docs/DECISIONS.md` when an architectural decision changed
5. The active Work Order closeout record
6. `Work-Order/CURRENT_WORK_ORDER.md`

## Required Documentation Outcomes

Each completed Work Order must produce exactly one truthful outcome for
every reviewed document:

- `UPDATED` — the document was changed to match repository truth
- `REVIEWED_NO_CHANGE` — it was reviewed and remains accurate
- `NOT_APPLICABLE` — the document is unrelated to the task
- `BLOCKED` — required documentation could not be updated safely

Do not update documents merely to create activity.
Do not rewrite unrelated sections.
Make the smallest documentation change needed to preserve accuracy.

## Required Knowledge Capture

Every Work Order must preserve the information necessary for a future
agent or maintainer to continue safely without relying on chat history.

Record material information such as:

- What behavior was added or changed
- Why the selected design was used
- Important constraints and invariants
- Configuration or schema changes
- New files, modules, or ownership boundaries
- Validation performed and its result
- Known limitations and remaining risks
- Decisions that must not be reopened without new evidence
- Deferred work that is explicitly outside the completed Work Order

Do not record temporary narration, speculative ideas, raw chain-of-thought,
or information already represented accurately elsewhere.

Prefer updating an existing canonical document over creating another
summary file.

## Documentation Stop Conditions

Stop closeout and report when:

- Implementation changed behavior but no authorized canonical document
  can be updated
- Two canonical documents conflict
- `docs/INDEX.md` does not identify authority for a material project area
- The active Work Order status disagrees with repository truth
- `CURRENT_WORK_ORDER.md` points to a completed, missing, or unauthorized
  Work Order
- Documentation would claim behavior not proven by validation

## Extended Completion Gate

A Work Order is complete only when:

- Acceptance criteria are satisfied
- Required tests and validation pass
- Git diff contains only authorized task files
- Documentation impact review is complete
- Canonical documents match implemented repository truth
- Important decisions, constraints, and risks are preserved
- `docs/INDEX.md` is accurate
- Work Order status is truthful
- `CURRENT_WORK_ORDER.md` is reconciled
- Final Git status satisfies the Work Order closeout policy

Code complete without documentation reconciliation is not Work Order
complete.

## Final Report Documentation Section

Every final report must include:

- `DOCUMENTATION_REVIEWED`
- `DOCUMENTATION_UPDATED`
- `DOCUMENTATION_REVIEWED_NO_CHANGE`
- `KNOWLEDGE_CAPTURED`
- `CURRENT_WORK_ORDER_STATUS`

Do not claim documentation closeout without inspecting the actual files.

## Project Traceability

Every material Work Order must maintain traceability from project
objective through capability, work order, files, validation, commit,
and current status to the next required gate.

### Before Implementation

The coding agent must identify:

1. Affected capability IDs from `docs/CAPABILITY_MATRIX.md`
2. Current status of each affected capability
3. Target status after this Work Order
4. The exact evidence required for the target status
5. Whether existing evidence already supports the target status

### Before Closeout

The coding agent must reconcile target versus actual status and update:

1. `docs/CAPABILITY_MATRIX.md` — update affected capability rows
2. `docs/VALIDATION_REGISTER.md` — add executed evidence rows
3. `docs/PROJECT_STATUS.md` — update status table and known risks
4. This section of `AGENTS.md` — record status-truth rules

### Status-Truth Rules

- A capability's status must never exceed what repository evidence
  proves.
- `TESTED` requires passing automated or bounded test evidence.
- `INTEGRATED` requires successful cross-component workflow validation.
- `LIVE_VERIFIED` requires representative use with Lightroom Classic
  and real project data.
- Code existence alone supports at most `IMPLEMENTED`.
- Focused automated tests alone support at most `TESTED`.
- Planned or expected work must not be recorded as completed work.
- A status downgrade (e.g., `INTEGRATED` → `TESTED`) is allowed when
  evidence shows the higher status is not yet justified.
- Unknown or unverifiable status must be `NOT_STARTED`, not guessed.

### Status-Truth Stop Conditions

Stop closeout and report when:

- A capability would need to be promoted beyond its evidence level.
- The register contains invented validation evidence.
- `CURRENT_WORK_ORDER.md` points to a completed Work Order after
  closeout.

## Read-First Invocation Rule

Every coding task must invoke the `project-read-first` skill before
implementation begins. This skill:

1. Resolves the canonical Git repository root.
2. Verifies Serena and CodeGraph project context.
3. Reads four mandatory authority documents completely.
4. Reads additional documents selectively based on the active Work
   Order's scope.
5. Produces a deterministic preflight report with exactly one terminal
   decision: `READY` or a `BLOCKED_*` reason.

The active Work Order's Required Read Order section defines the
mandatory read set; the Read-First skill defines the verification that
those reads completed correctly.

Do not skip the preflight even for small or obvious changes. The
preflight decision `READY` is required before any file modification.

---
> Source: [expellirmud-dot/Lightroom-AI-Workflow-](https://github.com/expellirmud-dot/Lightroom-AI-Workflow-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
