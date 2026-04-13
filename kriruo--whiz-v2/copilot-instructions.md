## whiz-v2

> Context is never static.

# Cursor Rules (for IDE Agent)

## 1. CORE PRINCIPLES

### 1.1 Living Context

Context is never static.

Before any major action, you must:

- Re-evaluate the current context
- Validate assumptions with the user
- Adjust your understanding
- Stop immediately if a conflict appears

### 1.2 Human-in-the-loop (Mandatory)

You must not act on unvalidated assumptions.

If something is unclear:

- Check best practices
- Check repo context
- For code language unclarities, visit Context7
- Ask the user

The user must explicitly approve:

- new assumptions
- breaking changes
- multi-file refactors
- deviations from plan
- uncertainty or ambiguity

### 1.3 Minimal Context Loading

Load only the files relevant to the task.

Do not scan the entire repo unless explicitly instructed.

Reuse stored context.

### 1.4 Code Quality

- Follow existing architecture and project patterns
- Keep code clean, modular, readable
- Add docstrings for new functions/classes
- Use inline comments only for non-obvious logic
- Do not over-refactor or rewrite unless asked

## 2. REPO CONTEXT MEMORY

Your project includes the directory:

`.ai/context/`

This directory contains project-specific knowledge.

Before beginning any task, you must:

Load relevant context files from:

- `.ai/context/project.md`
- `.ai/context/architecture.md`
- `.ai/context/conventions.md`
- `.ai/context/domains/*.md`
- `.ai/context/files/*.md`
- `.ai/context/tasks/*.md` (only if helpful)

If stored context conflicts with actual code:

- Actual code always wins
- Stop and ask the user whether to update context

## 3. WORKFLOW (STRICT)

Follow this workflow exactly.

### 3.1 Workflow Exceptions for Small Changes

For **small, isolated changes**, you may bypass the full workflow phases (0-4) and proceed directly to implementation if the task meets these criteria:

**You can bypass the workflow if:**
- **Simple, focused task** (single feature addition, bug fix, or refactor within a limited scope)
- **Few files affected** (typically 1-5 files with clear, obvious dependencies)
- **No breaking changes** (backward compatible, no API contract changes)
- **Clear requirements** (the change is straightforward and unambiguous)
- **Low risk** (change doesn't affect critical business logic or core architecture)

**Examples of tasks that can bypass:**
- Adding a new UI component following existing patterns
- Fixing a bug in a single component
- Adding a new API endpoint following existing patterns
- Updating styling or layout
- Adding utility functions
- Refactoring within a single module/component
- Updating documentation
- Fixing lint errors or type issues

**When bypassing the workflow, you must still:**

- Load the relevant file(s) to understand context
- Make the change following existing patterns
- Verify the change doesn't introduce errors (check lints)
- Briefly explain what was changed

**If ANY uncertainty arises during a "small change", immediately revert to the full workflow.**

### Phase 0 — Gather Context

**Step 0.1: Load repo context memory**

Load only relevant summaries.

**Step 0.2: Load relevant code files**

Only scan what is needed for the task.

**Step 0.3: Create a Concise Context Summary**

Produce:

```
## Context Summary

Goal:
Current behavior:
Expected behavior:
Files involved:
Dependencies:
Edge cases:
Patterns observed:
Risks / unknowns:
Critical assumptions (need validation):
```

**Step 0.4: Ask the user to validate**

Do not proceed until the user validates assumptions.

### Phase 1 — Plan

After user validation:

**Step 1.1: Re-check context validity**

Confirm no new commits, changes, or contradictions.

**Step 1.2: Impact Analysis**

```
## Impact Analysis

Files affected:
Breaking changes:
Cross-file implications:
Alternative approaches:
New assumptions (need validation):
```

**Step 1.3: Proposed Plan**

```
## Proposed Plan

1. Step…
2. Step…
3. Step…

Why this plan:
- Based on patterns…
- Based on business rules…
```

→ Ask for explicit user approval before implementing.

### Phase 2 — Execute

For each file to be modified:

**Step 2.1: Pre-Change Context Check**

```
## Pre-Change Check

- Code matches assumptions?
- Related files unchanged?
- Plan still valid?
```

If there's a discrepancy → stop and ask the user.

**Step 2.2: Implement**

- Follow validated plan
- Use existing patterns
- Keep changes minimal and targeted
- Pause if new uncertainty appears
- Tag inline questions with `// QUESTION:`

### Phase 3 — Test

**Step 3.1: Testing Framework (Pytest-First)**

- Default testing framework for this repo: **pytest**
- Preferred commands (unless the task clearly needs something more specific):
  - `python -m pytest tests/ -v`
  - Or targeted subsets (e.g. `python -m pytest tests/unit/test_settings_manager.py -v`)
- If pytest is not available or the project explicitly documents a different runner for a given task:
  - Ask the user which framework/command to use
  - Do not proceed with testing until the framework is confirmed

**Step 3.2: Run and interpret tests (using pytest by default):**

```
## Test Results

Testing framework used:
Unit tests:
Integration tests:
Manual scenario tests:
Side effects:
Regression checks:
Behavior matches validated assumptions?
```

If tests reveal unexpected behavior → stop and ask the user.

### Phase 4 — Update Context

After completing the task:

Update the appropriate context files under `.ai/context/`:

- file summaries
- domain summaries
- architecture notes (only if meaningful)
- task summary

Context updates must be:

- concise
- factual
- based on real code
- helpful for future tasks

Never store guesses.

## 4. WHEN TO ASK THE USER

You must pause and ask the user when:

- assumptions affect implementation
- breaking changes are involved
- context conflicts appear
- patterns or architecture are unclear
- multiple reasonable solutions exist
- stored context disagrees with code
- committing refactors, deletions, or new files

Do NOT ask when:

- the answer is trivial
- the user already answered
- it's clearly inferable from context
- the change is isolated and safe

Avoid repeating questions.

## 5. STYLE & QUALITY RULES

- Use clear, readable code
- Match formatting and lint rules
- Document new functionality
- Keep changes scoped
- Maintain project structure
- Resist rewriting unrelated areas

## 6. OUTPUT FORMAT

Every phase must output the correct structured format.

Never skip phases (except for small changes per section 3.1).

Never merge phases.

Never proceed without user validation at required gates (except for small changes per section 3.1).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KriRuo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
