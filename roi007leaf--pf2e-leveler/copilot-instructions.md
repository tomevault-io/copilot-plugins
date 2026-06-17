## pf2e-leveler

> These directives are written for Codex operating in this repository. They

# AGENTS.md - Codex Production Directives

These directives are written for Codex operating in this repository. They
override any default tendency toward shallow, fast, or incomplete output.

The governing loop for all work is: **gather context -> take action ->
verify work -> repeat**.

---

## 1. Pre-Work

### Step 0: Delete Before You Build

Before any structural refactor on a file larger than 300 lines, first
remove dead code: unused props, unused exports, unused imports, stale
helpers, and debug logs. If restructuring makes more code obsolete, remove
that too. Do not leave ghosts behind.

### Phased Execution

Do not attempt broad multi-file refactors in a single pass unless the user
explicitly asks for it. Break the work into phases. Keep each phase small
enough to reason about and verify properly. For larger changes, complete
Phase 1, verify it, report results, and wait for approval before moving to
the next phase.

### Plan and Build Are Separate Steps

If the user asks for a plan, or asks to think first, provide only the plan.
Do not write code until the user says to proceed.

If the user gives a written plan, follow it exactly. If you find a real
technical flaw in the plan, call it out clearly and stop for confirmation.
Do not silently improvise.

If the request is vague, do not start building. First describe what you
would build, where it belongs, and what tradeoffs matter.

### Spec-Based Development

For non-trivial work with multiple implementation decisions, first clarify
the contract. In Codex, use `request_user_input` when available in Plan
mode; otherwise ask concise direct questions in chat only when necessary.
Reduce ambiguity before editing code.

The spec is the contract. Execute against the agreed spec, not assumptions.

---

## 2. Understanding Intent

### Follow References, Not Descriptions

When the user points to existing code as a reference, inspect that code
first and match its conventions. Existing working code is a stronger spec
than a prose description.

### Work From Raw Data

When debugging, work from actual logs, stack traces, failing tests, and
repro steps. Do not guess. Trace the concrete failure.

If the bug report lacks raw error output and the failure cannot be derived
locally, ask for the console output or failing log directly.

### One-Word Mode

If the user says “yes”, “do it”, “push”, or similar after prior context has
already established the task, execute immediately. Do not restate the plan.

---

## 3. Code Quality

### Senior Dev Override

Do not settle for band-aids when the local design is clearly flawed. If the
change exposes duplicated state, inconsistent patterns, leaky abstractions,
or structural weakness, fix the underlying problem within scope and explain
the reasoning.

Ask: “What would a strong senior reviewer reject here?” Then address it.

### Forced Verification

Never report work as complete just because files were edited successfully.
Before closing the task, run all applicable verification that exists in the
repo:

- Type-checker / compiler
- Linter
- Tests
- Relevant manual validation or runtime checks

If one of these does not exist, say so explicitly. If one exists but cannot
be run, say why. Do not claim success with outstanding errors.

### Write Human Code

Write code that looks like an experienced human wrote it. Avoid noisy
commentary, decorative abstractions, and boilerplate explanations of obvious
logic.

### Don’t Over-Engineer

Do not design for hypothetical future requirements that the user did not
ask for. Prefer solutions that are simple, correct, and maintainable.

### Demand Elegance

For non-trivial work, pause and check whether the solution is merely working
or actually clean. If the first fix is obviously hacky, replace it with the
cleaner design before presenting it.

---

## 4. Context Management

### Use Delegation When It Helps

If the task spans many independent files or parallelizable subtasks, use
Codex sub-agents where appropriate. Give each sub-agent a narrow, concrete,
self-contained responsibility. Keep ownership boundaries clear so changes do
not conflict.

Do not delegate the immediate critical-path task if doing it locally is
faster and clearer.

### Context Decay Awareness

After a long conversation or after substantial time has passed, re-read any
file before editing it. Do not trust memory of file contents.

### Persistent State

Use the file system as durable memory when the task is long-running or
multi-step. If useful, maintain concise notes in files like
`context-log.md` or `gotchas.md` so future work can resume cleanly.

### File Read Discipline

Do not dump large files into context without need. Search first, then read
only the relevant sections. For very large files, inspect them in chunks.

### Tool Output Skepticism

If a search or shell result looks suspiciously incomplete, narrow the scope
and rerun it. Assume truncation or overly broad queries before assuming
absence.

---

## 5. File System as Working Memory

Use the file system actively instead of holding everything in chat context.

- Prefer targeted search over reading whole files.
- Save intermediate outputs when that makes debugging or verification more
  reliable.
- Use shell tools for filtering, searching, and inspecting project state.
- Preserve useful notes, decisions, and follow-up items in repo-local
  markdown files when that improves continuity.
- When debugging, keep reproducible logs or command outputs if they help
  validate the fix.

---

## 6. Edit Safety

### Edit Integrity

Before every edit, re-read the file. After every edit, read the affected
section again to verify the change landed correctly.

Do not make repeated blind edits against stale file contents.

### No Semantic Assumptions

Codex does not have guaranteed whole-program semantic awareness from a
single search. When renaming or changing a symbol, search separately for:

- Direct references and call sites
- Type references
- String literals containing the name
- Dynamic imports / requires
- Re-exports and barrel entries
- Tests, fixtures, and mocks

Assume one search pattern is insufficient.

### One Source of Truth

Do not solve rendering or state bugs by duplicating state. Keep one
authoritative source and derive everything else from it.

### Destructive Action Safety

Do not delete files until you verify nothing still references them. Do not
revert user changes unless explicitly asked. Do not push or perform other
shared-repo actions unless explicitly told to do so.

---

## 7. Codex Workflow Awareness

### Stay Within Codex’s Real Tooling

Do not write instructions that depend on unsupported Claude-specific
features or slash commands. Use the actual Codex toolset available in this
environment: shell commands, file reads, `apply_patch`, planning tools,
verification commands, and sub-agents when explicitly appropriate.

### Keep the Prefix Stable

Do not suggest changing models or tool availability mid-task unless the user
explicitly asks. Solve the task with the current environment.

---

## 8. Self-Improvement

### Mistake Logging

If the user corrects a recurring mistake or workflow issue, record the
lesson in `gotchas.md` when appropriate so the same mistake is less likely
to recur.

### Bug Autopsy

After fixing a bug, explain briefly why it happened and what would prevent
that category of bug in the future.

### Two-Perspective Review

For meaningful tradeoffs, present both views:

- What a perfectionist reviewer would still criticize
- What a pragmatist would accept as sufficient

Let the user choose when the tradeoff is real.

### Failure Recovery

If two fix attempts fail, stop and reassess. Re-read the relevant code
top-down, identify where the mental model was wrong, and explain the new
understanding before trying again.

### Fresh Eyes Pass

When testing your own change, evaluate it like a new user would. Flag rough
edges, confusing behavior, or missing validation.

---

## 9. Housekeeping

### Autonomous Bug Fixing

When given a concrete bug report, own the problem end-to-end. Trace the
failure, implement the fix, and verify it without asking the user to manage
the process for you unless you are blocked on missing external information.

### Proactive Guardrails

If a file is becoming hard to reason about, say so. If the repo lacks basic
validation, tests, or safety checks in the area you are touching, note that
once and propose the smallest useful guardrail.

### Batch Changes

When the same change must be applied across many files, group the work into
clear batches and verify each batch before moving on.

### File Hygiene

Prefer small, focused, navigable files. If a file has become unwieldy,
recommend splitting it along real responsibility boundaries.


<claude-mem-context>
# Memory Context

# [pf2e-leveler] recent context, 2026-05-31 9:33am GMT+3

Legend: 🎯session 🔴bugfix 🟣feature 🔄refactor ✅change 🔵discovery ⚖️decision 🚨security_alert 🔐security_note
Format: ID TIME TYPE TITLE
Fetch details: get_observations([IDs]) | Search: mem-search skill

Stats: 50 obs (17,914t read) | 293,866t work | 94% savings

### May 3, 2026
S834 Free Heart Background: Prompt for untrained skill replacement when background grants an already-trained skill (May 3 at 1:42 PM)
S120 Feat retraining implementation for pf2e-leveler — Task 2 complete (validator + apply), Task 3 (planner UI) in progress (May 3 at 1:42 PM)
### May 28, 2026
S835 Free Heart Background: Prompt for untrained skill replacement when background grants an already-trained skill (May 28 at 7:52 AM)
S836 Free Heart Background: Prompt for untrained skill replacement when background grants an already-trained skill (May 28 at 7:57 AM)
7061 1:36p 🟣 Full Bootstrap Suite Passes After Bidirectional Sync: 92/92 Tests, Lint Clean
7064 1:38p 🔵 CHANGELOG.md v3.4.31 Entry Is Incomplete — Does Not Reflect Full Dialog Overhaul Done This Session
7065 " ✅ Version Bumped to 3.4.32 — New Changelog Entry Added for Starting Skill Dialog Overhaul
7066 1:39p ✅ v3.4.32 Version Consistency Verified — Module Manifest Test Passes
7068 " ✅ All Changes Committed — Working Tree Clean at v3.4.32
7070 1:46p ⚖️ Reconsidering Whether Dedication Fallback Skills Count Toward 0/8 Starting Skill Limit
7071 " 🔵 importedFromActor.initialSkills Data Model Governs 0/8 Starting Skill Counter
7072 1:47p 🔴 Dedication Fallback Replacement Skills No Longer Count Against 0/8 Starting Skill Limit
7073 " 🔵 Test Failure: importedInitialSkillCount Still Includes Fallback Choice Skills After Refactor
7074 " 🔴 Dialog Live Count Fixed: Fallback-Locked Inputs Excluded from checkedCount
7075 " ✅ All Related Test Suites Pass After Dedication Fallback Skill Refactor
7076 1:48p ✅ CHANGELOG v3.4.32 Updated to Reflect Correct Fallback Skill Behavior
7077 " ✅ Full Test Suite Passes After Dedication Fallback Skill Refactor — 1474/1474
7079 1:49p ✅ pf2e-leveler Bumped to v3.4.33 with Dedication Fallback Skill Fix as Separate Release
7081 1:51p ✅ pf2e-leveler v3.4.33 Changes Fully Staged, All 1474 Tests Green
### May 31, 2026
7148 9:20a 🔵 pf2e-leveler FoundryVTT Module: Prerequisite & Feat Filtering Architecture
7149 " 🔵 pf2e-leveler Prerequisite Checker & Feat Picker Architecture: Deep Read
7150 9:21a 🔵 pf2e-leveler Additional Archetype Feat Unlock System & Feat Filter Category Logic
7152 " 🔵 Loremaster's Etude Feat Not Found in Local FoundryVTT Data
7153 " 🔵 pf2e-leveler Content Guidance System: GM-Controlled Per-Item Allow/Block
7155 9:22a 🔵 pf2e-leveler Build State Feat Computation: Alias System & Archetype Progress Tracking
7156 " 🔵 Level Planner Feat Selection Storage: Full Metadata Persisted on Pick
7157 9:23a 🔵 parsePrerequisiteNode("enigma muse") Parses as Feat Slug, Not Subclass Identity
7158 " 🟣 Added Failing Test: Loremaster's Etude Should Satisfy Enigma Muse Prerequisite via Loremaster Dedication Unlock
7159 " 🔵 Confirmed Bug: Loremaster's Etude "Enigma Muse" Prerequisite Fails for Non-Bard with Loremaster Dedication
7163 " 🔴 Fixed: Loremaster's Etude Enigma Muse Prerequisite Now Satisfied for Loremaster Dedication Unlocks
7164 9:24a 🔵 Fix Works But Test Assertion Too Strict: Prereq Text Now "Enigma Muse (via Loremaster Dedication)"
7165 " 🔴 Loremaster's Etude Enigma Muse Prerequisite Fix: Test Now Passing
7166 " 🔴 All 58 feat-picker Tests Pass and Lint Clean After Loremaster Fix
7167 " 🔴 Full Test Suite Passes After Loremaster Fix: 1484 Tests, 86 Suites, All Green
7173 9:25a ✅ pf2e-leveler v3.4.37: Working Tree Ready for Loremaster Fix Commit
7174 " ✅ CHANGELOG.md Updated for v3.4.37 with Loremaster Fix Entry
7181 9:27a 🔵 Bard Muse Subclass Aliases vs Loremaster Fix: Understanding the Full Picture
7183 " 🔵 Bard Class Definition in scripts/classes/bard.js; BARD Not Registered in build-state.test.js Suite
7184 9:28a ✅ Loremaster Fix Reverted from feat-picker.js: Searching for Better Approach
7185 " ✅ Loremaster Test Also Reverted from feat-picker.test.js: Full Rollback of First Approach
7186 " ✅ Full Rollback Complete: All Three Files Reverted to Pre-Fix State
7191 " 🟣 New Failing Test: Multifarious Muse rulesSelections Should Generate enigma-muse Alias in Build State
7192 " ✅ BARD Class Registered in build-state.test.js to Support Multifarious Muse Test
7194 " 🔵 Confirmed RED: feats.has('enigma') True but feats.has('enigma-muse') False for Multifarious Muse
7195 9:29a 🔵 getMultifariousMuseChoiceAliases Fix Not Working: getFeatChoiceSelectionMap May Not Read rulesSelections
7198 " 🔴 Multifarious Muse Choice Aliases Now Generated in Build State — Loremaster's Etude Bug Fixed at Root Cause
7199 " 🔴 Both Affected Test Suites Pass After Multifarious Muse Fix: 102 + 57 Tests Green
7201 9:30a 🔴 Full Suite Green: 1485 Tests, 86 Suites — Multifarious Muse Fix Complete and Verified
7202 " 🔴 pf2e-leveler v3.4.37: Multifarious Muse Fix Ready for Commit — Final State Verified
7203 9:31a ✅ Test Updated to Use flags.system.rulesSelections Instead of flags.pf2e.rulesSelections
7204 " 🔴 pf2e-leveler v3.4.37 Multifarious Muse Fix: Final Verification — 1485/1485 Tests Pass
7205 9:33a 🔵 pf2e-leveler Module Metadata: Author RoiLeaf, FoundryVTT 13-14 Compatibility
7206 " ✅ pf2e-leveler Version Bumped to 3.4.38 for Multifarious Muse Fix Release
7207 " 🔵 module-manifest.test.js Validates CI Release Workflow Version Stamping

Access 294k tokens of past work via get_observations([IDs]) or mem-search skill.
</claude-mem-context>

---
> Source: [roi007leaf/pf2e-leveler](https://github.com/roi007leaf/pf2e-leveler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
