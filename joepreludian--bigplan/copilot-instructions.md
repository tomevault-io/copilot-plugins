## bigplan

> Use when the user asks for a "big plan", "big delivery", a full feature, or any webapp scope that clearly decomposes into 5+ sub-specs with at least 2 backend and 2 frontend slices — orders them (backend first, then frontend, optionally DEVOPS), maintains a master index, and tracks which slices have been implemented via filename status. Make sure to use this skill whenever a request would produce 5+ deliverable specs spanning both an API/database and a UI, when the user mentions decomposing a large webapp feature, or any time scope clearly exceeds a single brainstorming session — even if they don''t say "bigspec". Skip this skill for smaller scopes — a single-tier change, fewer than 5 total slices, or fewer than 2 backend or 2 frontend slices go straight to superpowers:brainstorming. Pairs with superpowers:brainstorming and superpowers:writing-plans to author each sub-spec.


# bigspec — Orchestrating Big Webapp Deliveries

## Purpose

Big webapp features die when you try to spec them as one document. The backend, the API surface, the data model, the frontend pages, the wiring — bundled together they produce a spec that's either too vague to execute or too long to review. `bigspec` solves this by **decomposing the big idea into an ordered list of small, self-contained sub-specs**, each of which is itself authored using the regular `superpowers` flow (`brainstorming` → `writing-plans`).

You are the **orchestrator**. You don't write the small specs from scratch — `superpowers:brainstorming` does. You decide *what* the slices are, *what order* they ship in, and you keep the master index honest about which ones are done.

## When to engage

bigspec has a **hard size threshold**. Engage only when **all three** of the following hold:

1. The work decomposes to **5 or more sub-specs** total.
2. At least **2 of those slices are backend** (data model, API, jobs, auth, infra-adjacent).
3. At least **2 of those slices are frontend** (pages, components, state, API client).

If the work doesn't clear that bar, don't use bigspec — go straight to `superpowers:brainstorming`. bigspec adds index-keeping overhead that only pays off when there are enough slices to coordinate.

Other strong signals that you're in bigspec territory (subject to the threshold above):

- The user says "big plan", "big delivery", "big feature", "the whole flow", or anything that signals a larger-than-one-spec scope.
- The user lists many distinct deliverables in the same request.
- A first attempt at `superpowers:brainstorming` reveals the spec covers multiple independent subsystems — back out and use bigspec instead.

### How to check the threshold

If you're uncertain whether the request clears the threshold, do a quick **slice sketch in your head** before committing: list the deliverables you'd expect, tag each as BE/FE/DEVOPS, count. If you can't reach 5 with at least 2 BE and 2 FE, drop bigspec and use `superpowers:brainstorming` instead. Don't pad slices to clear the bar — that produces fake decomposition that wastes everyone's time.

If the count is genuinely close to the threshold, ask the user briefly: "I'm sizing this — sounds like roughly N backend pieces and M frontend pieces. Does that match how you're thinking about it?" Their answer settles it.

## Process overview

```
1. Capture the big picture            (you, with the user)
2. Decompose into slices              (you, with the user)
3. Write master index + TOPLAN files
   one TOPLAN file per slice with a starter prompt for brainstorming  (you)

   ─── SPEC PHASE (brainstorm) ───────────────────────────────
4. For each slice, in order:          (TOPLAN → BRAINSTORMED)
     brainstorm the starter prompt    (superpowers:brainstorming)
     rename TOPLAN → BRAINSTORMED
     emit progress: "X of Y specs brainstormed"
   Gate: do not proceed to plan phase until X == Y.

   ─── PLAN PHASE (writing-plans) ────────────────────────────
5. For each BRAINSTORMED slice:       (BRAINSTORMED → AVAILABLE)
     ask user: run plans in parallel? (default: yes — safe)
     write the plan                   (superpowers:writing-plans)
     rename BRAINSTORMED → AVAILABLE
     emit progress: "X of Y plans written"
   Gate: do not proceed to implementation until X == Y.

   ─── IMPLEMENTATION PHASE ──────────────────────────────────
6. For each AVAILABLE slice:          (AVAILABLE → IMPLEMENTED)
     default sequential, in slice number order
     parallel only if user asks AND parallel-safety check passes
     rename AVAILABLE → IMPLEMENTED   (you, post-merge)
```

You move through this with the user — not in one shot. After step 3 the master index becomes the source of truth and you return to it between phases.

### The two gates

bigspec has **two named gates** that separate the three phases. You will refer to them by name when narrating progress to the user — saying "we're at the spec-phase gate" is faster and clearer than "we've finished step 4 but not started step 5".

- **Spec-phase gate** — between Step 4 (brainstorming) and Step 5 (plan-writing). The gate is closed until **every** slice is `BRAINSTORMED`. While the gate is closed, you must not write any plan, even for slices that are already brainstormed. The gate opens when `X of Y specs brainstormed` reads `Y of Y`.
- **Plan-phase gate** — between Step 5 (plan-writing) and Step 6 (implementation). Closed until **every** slice is `AVAILABLE`. While the gate is closed, you must not implement any slice. Opens when `X of Y plans written` reads `Y of Y`.

Whenever you advance a slice's status, end the message with a one-line "gate state" so the user sees where you are: *"Spec-phase gate: 4 of 7 — closed."* / *"Spec-phase gate: 7 of 7 — open. Plan-phase gate: 0 of 7 — closed."*

These gates are non-negotiable. They are the structural reason this skill exists; the format below depends on them being held.

### Why spec-first, then plan, then implement

Specifying every slice before writing any plan, and writing every plan before any implementation, catches the worst category of bug a big delivery can have: **a decomposition mistake**. While brainstorming slice 03 you'll routinely discover that slice 01's data model is missing a field, or that slice 04's API contract conflicts with slice 02's job semantics. Fixing those at the spec stage costs minutes; fixing them after slice 01 ships costs days and a migration.

The same applies to plans: writing all plans before any implementation gives the user a complete, end-to-end blueprint of what's about to be built, and lets them spot a wrong abstraction before the first line of production code is written.

Once every slice is `AVAILABLE`, multiple engineers (or subagent fleets) can pick slices off the index independently because all contracts and plans are pinned down.

## Context hygiene — clean before each expensive step

A big delivery's specs, plans, and code easily exceed the orchestrator's working context. Reading 8 design docs, 8 plans, and an INDEX.md back-to-back will saturate the window long before the work is done. The skill is **only correct when the orchestrator stays small** — large context produces fuzzy decisions, missed file overlaps, and lost slice numbers.

The disk is the source of truth. Every slice's status is encoded in its filename; every commitment is in `INDEX.md`. Clearing context loses nothing — when the orchestrator picks up the next step, it re-reads only the files it needs.

**Before each of the following steps, clean context first:**

- Before each Step 4a brainstorm.
- Before each Step 5b plan-writing run (per slice, if running sequentially; per batch, if running in parallel).
- Before each Step 6c implementation.

**How to clean:**

1. **Preferred — dispatch as a subagent.** Subagents get a fresh context window. Brief them with: the path to `INDEX.md`, the path to the relevant slice file(s), and the exact step to run. The orchestrator's context stays small because the subagent's work happens in its own context, not yours. This is the default for `superpowers:brainstorming`, `superpowers:writing-plans`, and `superpowers:subagent-driven-development`.
2. **Fallback — ask the user to `/compact`.** If the user is running steps inline in the main session (no subagents), pause before the next expensive step and ask: *"About to start the next slice. I'd like to compact my context first so I don't carry the previous slice's spec into this one — please run `/compact` if you can, then say 'go' and I'll re-read the index and continue."* After compacting, the orchestrator re-reads `INDEX.md` and the specific slice file(s) it needs and proceeds.

**Always commit before clearing.** Anything not on disk and committed is lost. The skill's commit cadence (after every status flip) exists exactly so context can be cleaned between any two steps without losing progress.

**What to re-read after a clean:**

- Always: `docs/bigspec/<topic>/INDEX.md` (the orchestration state).
- For brainstorming a slice: that slice's TOPLAN file.
- For plan-writing a slice: that slice's BRAINSTORMED file.
- For implementing a slice: that slice's AVAILABLE file AND the linked plan file.

Do **not** re-read every slice in the directory. The whole point is to stay small. Read only the files the immediate step needs.

---

## Step 1 — Capture the big picture

Have a short conversation to nail down four things. Ask one at a time; multiple choice is welcome. Don't skip ahead to decomposition until these are clear.

1. **What is the user-facing outcome?** One or two sentences. ("Logged-in users can create and share project boards.")
2. **What is explicitly out of scope?** This is more important than what's in scope — it bounds the slices.
3. **What constraints exist?** Existing schemas, auth model, deploy cadence, frameworks. Read the codebase if needed.
4. **What does "done" look like for the whole delivery?** A demo-able flow, a metric, a feature flag flipped, etc.

These answers go into the master index `## Context` section verbatim. Don't paraphrase — they're the brief that every slice will be authored against.

## Step 2 — Decompose into slices

A slice is a sub-spec. **One slice = one self-contained spec = one cycle of brainstorming → plan → implement.**

### Slicing rules

- **Backend slices first, frontend slices after, DEVOPS slices wherever they unblock the others.** Frontend is much easier to spec well when the API it calls already exists (or has a written, agreed contract). Going backend → frontend means each frontend slice is a thin layer on top of something concrete. DEVOPS slices (CI changes, infra, deploy config, feature-flag plumbing) usually go just before the slices they unblock.
- **A slice is shippable on its own.** "Shippable" doesn't mean user-visible — a backend slice can ship behind a flag with no UI. It means the slice produces working, tested code that doesn't break the build.
- **Each slice has one obvious owner-file or owner-area.** If you can't name the primary files a slice will touch, it's not sliced finely enough.
- **Slices are roughly equal in size.** Aim for slices that fit one focused implementation session. If one slice dwarfs the others, split it.
- **No forward dependencies.** Slice N must not require code from slice N+k. If a frontend slice depends on a yet-to-be-built endpoint, the endpoint slice must come first.

### Tiers and numbering

Sub-specs are numbered so the directory sorts in execution order. The tier is encoded in the filename (see "Slice filename" below) so a glance at the directory tells you what each slice is:

- `01–09` → **backend slices** (BE) — data model, migrations, API endpoints, jobs, auth
- `10–19` → **frontend slices** (FE) — pages, components, state, API client wiring
- `20+` → **devops / cross-cutting** (DEVOPS) — CI, infra, deploy config, observability, feature-flag rollout cleanup

Pad to two digits even if you only have a few slices. This keeps room for inserting a slice later (`05a-`, or renumber).

### Slice filename

Each slice file is named:

```
NN-{AVAILABLE|IMPLEMENTED}-{BE|FE|DEVOPS}-YYYY-MM-DD-<kebab-name>.md
```

Where:

- `NN` — two-digit slice number, defines execution order.
- `AVAILABLE` / `IMPLEMENTED` — lifecycle status. Newly-written slices are `AVAILABLE`. After the slice's code is merged, **rename the file** to swap `AVAILABLE` → `IMPLEMENTED`.
- `BE` / `FE` / `DEVOPS` — tier identifier; matches the numbering range.
- `YYYY-MM-DD` — date the slice spec was first written. This date does **not** change when the file is renamed to `IMPLEMENTED`; it remains the spec's authorship date.
- `<kebab-name>` — ~3-word, obviously distinguishable from siblings.

Examples (`docs/bigspec/project-boards/`):

```
INDEX.md
01-AVAILABLE-BE-2026-05-02-board-data-model.md
02-AVAILABLE-BE-2026-05-02-board-crud-api.md
03-AVAILABLE-BE-2026-05-02-board-share-tokens.md
04-AVAILABLE-BE-2026-05-02-board-digest-job.md
10-AVAILABLE-FE-2026-05-02-board-list-page.md
11-AVAILABLE-FE-2026-05-02-board-editor-page.md
12-AVAILABLE-FE-2026-05-02-public-share-view.md
20-AVAILABLE-DEVOPS-2026-05-02-flag-rollout-config.md
```

After the data-model slice is implemented and merged, that single file is renamed to:

```
01-IMPLEMENTED-BE-2026-05-02-board-data-model.md
```

The numeric prefix preserves chronological order; the AVAILABLE/IMPLEMENTED token gives an immediate visual signal of what's left to ship.

### Validate the slicing with the user

Before you write the index, present the proposed slices in a single message — numbered list, one line each — and ask the user to confirm or edit. This is the single most leveraged moment in the whole skill: a wrong slice list multiplies waste across every later step.

## Step 3 — Write the master index AND a TOPLAN file per slice

This is the moment everything gets committed to disk. You produce two artifacts:

### 3a. The master index

Create `docs/bigspec/<topic>/INDEX.md` using the template at `references/spec-index-template.md`. The index has three jobs:

1. **Brief** — the four answers from Step 1, so every slice's brainstorming session can read the same context.
2. **Slice list with status** — a row per slice showing where it is in the lifecycle.
3. **Open questions** — anything you and the user couldn't resolve up front and want to revisit when the relevant slice comes up.

The slice list uses these statuses (filename status token in parentheses):

- `toplan` (`TOPLAN`) — starter prompt written; not yet brainstormed
- `brainstormed` (`BRAINSTORMED`) — full design doc written via `superpowers:brainstorming`; no implementation plan yet
- `available` (`AVAILABLE`) — implementation plan written via `superpowers:writing-plans`; ready to implement
- `implementing` — code in progress
- `done` (`IMPLEMENTED`) — merged; file renamed `AVAILABLE` → `IMPLEMENTED`

### 3b. One TOPLAN file per slice

For every slice in the table, create a file at:

```
docs/bigspec/<topic>/NN-TOPLAN-{BE|FE|DEVOPS}-YYYY-MM-DD-<kebab-name>.md
```

Each TOPLAN file is a **comprehensive starter prompt** for `superpowers:brainstorming`. Its job is to give that future brainstorming session everything it needs to author a high-quality spec without bouncing context back to bigspec. Use the template at `references/toplan-template.md`. At minimum, each TOPLAN file must contain:

- A pointer to the parent `INDEX.md` (so brainstorming can read the shared brief)
- A one-paragraph **scope** statement: what this slice produces
- An explicit **non-scope** list: what this slice does NOT touch (often things owned by adjacent slices)
- **Inputs / dependencies**: which earlier slices' contracts this one builds on (by slice number — never by code that doesn't exist yet)
- **Outputs / contracts**: what later slices will rely on (API shape, DB columns, event names)
- **Open questions** specific to this slice that brainstorming should resolve
- A **success criterion** in one sentence: how we'll know this slice is correctly specified

Write all the TOPLAN files in the same pass — don't pipeline. Doing them all at once forces you to keep the dependency graph between slices consistent.

### 3c. Commit

Commit the index plus all TOPLAN files together as a single atomic commit. The repo state from this point is: "decomposition done, ready to brainstorm." Anyone reading the directory can see exactly what's planned and start working any TOPLAN slice.

## Step 4 — Spec phase: TOPLAN → BRAINSTORMED for every slice

Iterate through the TOPLAN files in number order. For each one:

### 4a. Brainstorm the starter prompt

> **Clean context first** (see "Context hygiene" above). Verify INDEX.md and the slice's TOPLAN file are committed; then either dispatch the brainstorm as a subagent (preferred — its context is fresh by construction) or ask the user to `/compact` before continuing inline.

Invoke `superpowers:brainstorming` and hand it:

- The path to the slice's TOPLAN file (so it reads the starter prompt verbatim).
- The path to the parent `INDEX.md` (for shared context).
- A clear instruction that the resulting design doc must be written **back to the same slice file**, replacing the TOPLAN starter content with the full spec.

Tell the user up front, on its own line:

> "Brainstorming slice NN of Y. Starter at `<toplan path>`. Output will replace the file in place."

When brainstorming completes, the file's content is now a real design doc. **Rename the file** by swapping `TOPLAN` → `BRAINSTORMED`:

```
NN-TOPLAN-{tier}-YYYY-MM-DD-<name>.md
  →  NN-BRAINSTORMED-{tier}-YYYY-MM-DD-<name>.md
```

Update the slice's status in `INDEX.md` from `toplan` to `brainstormed`. Commit the rename and index update together.

### 4b. Emit progress line

After each successful flip to `BRAINSTORMED`, output a single progress line in the conversation:

> **`X of Y specs brainstormed`**

Where `X` is the count of slices at status `BRAINSTORMED` or further (`AVAILABLE`, `IMPLEMENTED`) and `Y` is the total slice count. Examples:

- After the first slice's brainstorming: `1 of 7 specs brainstormed`
- After the last: `7 of 7 specs brainstormed — ready for plan phase`

Give it its own line, in code or bold formatting. Don't bury it in a paragraph.

### 4c. Hold the spec-phase gate until X == Y

The **spec-phase gate** stays closed until every slice is `BRAINSTORMED`. While it's closed, do not start the plan phase, do not write any plans, do not write any production code. The gate opens automatically when the progress line reads `Y of Y specs brainstormed`. State the gate's status in the message after each progress line update.

If, during the spec phase, brainstorming a slice surfaces a problem with an earlier `BRAINSTORMED` slice (e.g. slice 04 needs a field that slice 01's spec didn't include), **revise the earlier spec right then**. The cost is a quick edit; the cost of letting it slide is far higher. If the revision is structural (e.g. a slice splits in two), update the index and add a row to its change log.

If brainstorming a slice surfaces that a slice was missing entirely, insert a new TOPLAN file (using the same template) and bump `Y`. The progress line now reads `X of Y+1`. Don't try to keep `Y` constant — the truth that the decomposition grew is exactly what you want visible.

## Step 5 — Plan phase: BRAINSTORMED → AVAILABLE for every slice

This phase runs `superpowers:writing-plans` against each `BRAINSTORMED` slice. The output of plan-writing is a plan file at `docs/superpowers/plans/YYYY-MM-DD-<slice-name>.md`, after which the slice file is renamed `BRAINSTORMED` → `AVAILABLE` and the index is updated with the plan's path.

### 5a. Ask the user about parallelism

Plan-writing is **safe to parallelize**. Each plan reads its slice's design doc plus the master index for shared context, but no plan modifies code, no plan depends on another plan's output, and the design doc contracts were already pinned during the brainstorm phase. So multiple writing-plans subagents can run concurrently without stepping on each other.

Before starting Step 5, ask the user:

> "Plan phase: I have **N slices** to plan. Each one is a `superpowers:writing-plans` invocation against an already-brainstormed spec. They're independent — slices reference each other's contracts but don't modify each other. I can run them:
>
> 1. **In parallel** (recommended) — N subagents at once, faster but uses more tokens concurrently.
> 2. **Sequentially** — one at a time, slower but predictable token burn.
>
> Which do you prefer?"

Default to parallel if the user accepts. If `N` is very large (e.g. 15+), warn that parallel will burst a lot of tokens and offer to batch (e.g. groups of 5).

### 5b. Run writing-plans (parallel or sequential)

> **Clean context first** (see "Context hygiene" above). In parallel mode, each writing-plans subagent has fresh context by construction — no extra cleanup needed beyond ensuring everything is committed before dispatch. In sequential mode, clean between each slice (subagent dispatch or `/compact`) — otherwise plan N's writing carries plan N-1's design doc forward.

For each `BRAINSTORMED` slice, dispatch `superpowers:writing-plans` with:

- Path to the slice's `BRAINSTORMED` file (the design doc).
- Path to the parent `INDEX.md`.
- The plan's expected output path: `docs/superpowers/plans/YYYY-MM-DD-NN-<kebab-name>.md` (use today's date, prefix with the slice number so plan files sort the same way slices do).

If running in parallel, dispatch all subagents in a single batch. If sequential, dispatch one at a time, awaiting each before the next.

When a plan completes, **rename the slice file** `BRAINSTORMED` → `AVAILABLE`:

```
NN-BRAINSTORMED-{tier}-YYYY-MM-DD-<name>.md
  →  NN-AVAILABLE-{tier}-YYYY-MM-DD-<name>.md
```

Update the index's slice row: status flips to `available`, plan column gets the relative path to the plan file. Commit each (slice rename + index update + plan file) together.

### 5c. Emit progress line

After each successful flip to `AVAILABLE`, output:

> **`X of Y plans written`**

Where `X` is the count of slices at status `AVAILABLE` or further. Same formatting rules as Step 4b.

In parallel mode, multiple plans may finish in quick succession; emit a progress line for each as they complete, in the order they finish. The user sees the count climb as agents return.

### 5d. Hold the plan-phase gate until X == Y

The **plan-phase gate** stays closed until every slice is `AVAILABLE`. While it's closed, do not start any slice's implementation. The gate opens when the progress line reads `Y of Y plans written`. State the gate's status in the message after each progress line update.

If, while writing a plan, the planning subagent discovers a real flaw in the design doc (not just a small clarification), it should pause and surface the issue. Revise the spec, possibly reset that one slice back to `BRAINSTORMED`, and re-plan. Don't let a known-bad plan land just because it was written.

## Step 6 — Implementation phase: AVAILABLE → IMPLEMENTED

Only enter this phase once every slice is `AVAILABLE`.

### 6a. Ask the user about parallelism (and audit if so)

Implementation is **NOT safe to parallelize by default**. Multiple slices simultaneously editing the same shared file (e.g. `package.json`, `schema.rb`, `app/router.tsx`, a migrations directory, a CI config) will produce merge conflicts and broken builds.

Default: **sequential**, in slice number order.

Before starting Step 6, ask the user:

> "Implementation phase: **N slices** to implement. Default is sequential (slice 01 → 02 → … in order). I can run multiple in parallel only if (a) you ask for it AND (b) a file-overlap audit shows no two parallel slices touch the same file. Want me to (1) run sequentially, or (2) audit for parallelism and tell you what's safely parallelizable?"

### 6b. Parallel-safety audit (only if user asks)

If the user opts into parallelism, perform the audit at `references/parallel-implementation-checklist.md`. The audit produces three buckets:

1. **Safe-to-parallelize** — slices whose plans touch a fully disjoint set of files from every other slice in this bucket.
2. **Must-be-sequential** — any slice that touches a file also touched by another `AVAILABLE` slice. Conflicting slices must run one at a time.
3. **Borderline / ambiguous** — slices where the plan can't enumerate exact files (e.g. "modify the relevant route registry") and conflict can't be ruled out. Treat as must-be-sequential by default; ask the user if you're not sure.

Present the audit result to the user before launching any subagent. Token-cost considerations: if the safe-to-parallelize bucket has many slices, the burst could be large; warn the user if it looks expensive.

**File-overlap is a hard line, not a heuristic.** If two slices both modify any single file (even one line each), they cannot run in parallel. There is no version of "small enough to merge by hand" that's worth the risk in this skill.

### 6c. Implement each slice

> **Clean context first** (see "Context hygiene" above). Implementation is the heaviest step in token cost — code reading, edits, tests. ALWAYS dispatch implementation as a subagent (preferred) so its context is independent of the orchestrator's. If the user insists on inline implementation, ask them to `/compact` before each slice.

For each slice (or each batch in the safe-to-parallelize bucket), hand off to whichever execution path the user prefers (`superpowers:subagent-driven-development`, `superpowers:executing-plans`, or manual). Once a slice has a plan, this is a normal superpowers implementation task.

Update slice status to `implementing` when a slice's implementation starts, and back to `available` only if implementation is abandoned (rare — usually it just goes to `done`).

### 6d. Flip AVAILABLE → IMPLEMENTED

When the slice's code is merged (or otherwise considered done), **rename the slice file** by swapping the status token:

```
NN-AVAILABLE-{tier}-YYYY-MM-DD-<name>.md
  →  NN-IMPLEMENTED-{tier}-YYYY-MM-DD-<name>.md
```

Everything else in the filename — the number, the tier, the authoring date, the kebab-name — stays the same. The date does **not** advance to the merge date; it remains the spec's authorship date so the historical record is preserved.

This is the durable signal that the slice is done — independent of git history, branches, or the index file. Anyone reading the directory sees the lifecycle of every slice at a glance. Update the index status to `done` and commit both changes together.

---

## Working alongside superpowers

`bigspec` does not replace any superpowers skill. Concretely:

- **`brainstorming`** still owns the design conversation for each slice. bigspec just decides which slice is being brainstormed next and where its output file lives.
- **`writing-plans`** still owns the plan format and content. bigspec just notes the plan path in the master index.
- **`executing-plans` / `subagent-driven-development`** are unchanged.

If a brainstorming session reveals that a slice is actually two slices, that's fine — go back to the index, split the slice, renumber if needed, and continue. The index is editable.

If, mid-delivery, the user adds new scope, **do not silently grow existing slices**. Add new slices to the index (with appropriate numbering) and brainstorm them in turn.

## When *not* to use bigspec

- Single-deliverable requests ("add an endpoint to return X", "fix this bug", "rename this column").
- Pure refactors with no new feature surface.
- Spike/exploration work where the answer is "find out if X is feasible" — those are ad-hoc, not spec-driven.
- Anything where the user has already produced a spec or plan and just wants implementation help.

In those cases, bypass bigspec and use the standard superpowers flow (or just do the work).

## Reference files

- `references/spec-index-template.md` — the master index template you fill in at Step 3a.
- `references/toplan-template.md` — the TOPLAN starter-prompt template you fill in once per slice at Step 3b.
- `references/sub-spec-conventions.md` — slice-naming, numbering, and renumbering details if the simple cases above don't cover your situation.
- `references/parallel-implementation-checklist.md` — the file-overlap audit you run before parallelizing implementation in Step 6b.

---
> Source: [joepreludian/bigplan](https://github.com/joepreludian/bigplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
