## midtempo-framework

> Last Updated: 22/04/2026

# Agent Rules

Last Updated: 22/04/2026

**Agent's Core Principle:** Do the simplest thing that works.

---

## Table of Contents

- [Pre-Action Gate](#0-pre-action-gate)
  - [Step 1: Route to Appropriate Skill](#step-2-route-to-appropriate-skill)
  - [Step 2: Enforce Skill-Based Code Changes](#step-3-enforce-skill-based-code-changes)
- [Quick Reference](#1-quick-reference)
- [Iron Laws](#2-iron-laws)
- [Workflow](#3-workflow)
- [Skill Router](#4-skill-router)
  - [Skills](#41-skills)
  - [Task Skills](#42-task-skills)
  - [Review Skills](#43-review-skills)
  - [Routing Rules](#44-routing-rules)
- [Writing Style](#5-writing-style)
- [Tooling](#6-tooling)
  - [Core Tooling](#61-core-tooling)
  - [Additional Tooling](#62-additional-tooling)
  - [Tooling Rules](#63-tooling-rules)
- [Bypass Protection](#6-bypass-protection)
- [Stop Conditions — ASK HUMAN](#7-stop-conditions--ask-human)

---

## 0. Pre-Action Gate

### Step 1: Route to Appropriate Skill

**REQUIRED ACTION:**
```
SCAN user request for trigger words (see "§4. Skill Router")
SCAN SCOPE: includes the content of any referenced or attached documents, not just the user's typed message.

IF referenced document contains a **Skill:** field
  → Treat as a direct skill routing instruction
  → READ this ENTIRE file (CLAUDE.md) first
  → THEN read the skill file specified in the **Skill:** field
  → FOLLOW skill instructions exactly

IF TRIGGER WORDS FOUND:
  → READ this ENTIRE file (CLAUDE.md) first
  → THEN read the matching skill file
  → FOLLOW skill instructions exactly

IF NO TRIGGER WORDS FOUND:
  → Proceed to "Step 3: Enforce Skill-Based Code Changes"
```

### Step 2: Enforce Skill-Based Code Changes

**MANDATORY:** Before making any code changes

**REQUIRED ACTION:**
```
IF request involves changing production code, tests, or configuration:
  AND no `midtempo-framework/*.md` skill is active:
    → STOP immediately
    → DO NOT make code changes
    → ASK user which skill to run
    → EXPLAIN: "Code changes require an midtempo-framework skill (build, bug, refactor, refine)"
    → WAIT for user to specify skill or approve ad-hoc change

IF simple non-code tasks (documentation only, research, file reading):
  → VALID: Proceed with task
```

**MANDATORY RULE:** NEVER MAKE CODE CHANGES WITHOUT AGENTIC-FRAMEWORK SKILL

---

## 1. Quick Reference

- **Test Driven Development always** — never production code without a failing test.
- **Files ≤ 500 lines, functions ≤ 75 lines**
- **UK English** throughout
- **No `console.log`** — only `.warn`/`.error` 
- **No disabling lint**  without human approval
- **Coverage thresholds:** lines 90%, functions 90%, branches 70%
- **Test output:** `/planning/last-test-ran.log`
- **Commit messages:** No Attribution or Co-Authored-By tags

---

## 2. Iron Laws

These rules cannot be waived. Skill files enforce them; violations trigger automatic recovery.

| Law        | Rule                                     | Skill                      |
| ---------- | ---------------------------------------- | -------------------------- |
| TEST DRIVEN DEVELOPMENT (TDD)        | No production code without failing test  | `/midtempo-framework/rules/tdd.md`     |
| BUGS       | Trace to root cause, never fix symptoms  | `/midtempo-framework/bugs.md`          |
| REFACTOR   | Only on green, behaviour unchanged       | `/midtempo-framework/refactor.md`      |
| TESTING    | Test real behaviour, not mocks           | `/midtempo-framework/rules/testing.md` |
| COMPLETION | No placeholders, implement fully or omit | —                          |

Note: All Dates in DD/MM/YYYY format.

---

## 3. Workflow

All feature work follows this sequence. No phase may be skipped. Human approval required at each step.

```
BUILD → DESIGN DOC → PLAN DOC → TEST DOC → RED → GREEN → REFACTOR → UPDATE DOCS
```

| Phase         | Skill                      | Output                        |
| ------------- | -------------------------- | ----------------------------- |
| Build    | `/midtempo-framework/build.md` | `planning/[feature]-decisions.md` |
| Design    | `/midtempo-framework/write-design.md` | `planning/[feature]-design.md` |
| Plan          | `/midtempo-framework/write-plan.md`    | `planning/[feature]-plan.md`   |
| Test Manifest | `/midtempo-framework/write-tests.md` | `planning/[feature]-tests.md`  |
| Red → Green → Refactor → Docs    | `/midtempo-framework/deliver.md`       | Working code, tests, docs     |

---

## 4. Skill Router

**Before responding to any request, identify the matching skill and read it FIRST.**

Skills do not stack. If multiple triggers match, clarify which skill applies.

### 4.1 Skills

These drive the delivery pipeline. Each produces artefacts for the next phase.

| Trigger Words                      | Skill                      | Purpose                         | Output                        |
| ---------------------------------- | -------------------------- | ------------------------------- | ----------------------------- |
| setup     | `/midtempo-framework/setup.md` | Setup the midtempo-framework for the repo      | `/midtempo-framework/instructions/*.md` files |
| idea, brainstorm, build, explore, create     | `/midtempo-framework/build.md` | Transform idea into design      | `planning/[feature]-design.md` |
| write plan, delivery plan, plan          | `/midtempo-framework/write-plan.md`    | Design into implementation plan | `planning/[feature]-plan.md`   |
| write tests, test manifest, tests            | `/midtempo-framework/write-tests.md`    | Create our test manifest | `planning/[feature]-tests.md`   |
| phase, run phase, deliver, delivery | `/midtempo-framework/deliver.md`       | Execute plan via TDD phases     | Working code, tests, docs     |

### 4.2 Task Skills

These handle specific work types outside the main workflow.

| Trigger Words                         | Skill                    | Purpose                           | Prerequisite      |
| ------------------------------------- | ------------------------ | --------------------------------- | ----------------- |
| bug, broken, error, failing, issue, problem    | `/midtempo-framework/bugs.md`        | Trace to root cause, fix          | None              |
| investigate, investigation, unclear, look into, why  | `/midtempo-framework/investigate.md` | Deep discovery, recommendations   | None              |
| refactor, clean, simplify, tidy       | `/midtempo-framework/refactor.md`    | Improve structure, keep behaviour | Tests green       |
| refine, tweak, fix                 | `/midtempo-framework/refine.md`   | Refine delivered feature          | Design doc exists |
| build e2e, e2e tests, write e2e, end-to-end | `/midtempo-framework/build-e2e.md` | Build E2E tests post-delivery     | Design doc exists |

### 4.3 Review Skills

These evaluate work quality. All require planning docs context (Gate 0).

| Trigger Words       | Skill                            | Scope                              |
| ------------------- | -------------------------------- | ---------------------------------- |
| review code         | `/midtempo-framework/review-code.md`         | Logic, correctness, standards      |
| review tests        | `/midtempo-framework/review-tests.md`        | Test quality, behavioural coverage |
| review architecture | `/midtempo-framework/review-architecture.md` | Boundaries, complexity, coupling   |
| review e2e          | `/midtempo-framework/review-e2e.md`          | E2E reliability, element selection |

### 4.4 Routing Rules

1. **Exact match:** Use the skill matching the trigger words
2. **No match:** Check if request implies a workflow phase (§3)
3. **Ambiguous:** Ask which skill applies before proceeding
4. **Multiple tasks:** Handle sequentially, one skill at a time

---


## 5. Writing Style

**Read:** `/midtempo-framework/rules/writing.md` and output the declaration gate.

---

## 6. Tooling

**Tooling Law:** Commands are immutable permission patterns. Exact invocation required — modifications (pipes, redirections, extra flags) void auto-approval. Violations trigger manual review gates. Rationale in §6.2.

### 6.1 Core Tooling

#### Testing
```bash
npm run test:python    # Run all Python tests
npm run test:python:unit    # Run Python unit tests only (fast, no external dependencies)
npm run test:python:integration    # Run Python integration tests only (slower, may use external resources)
npm run test:python:coverage    # Run Python unit tests with coverage report
```

#### Linting
```bash
npm run lint:python    # Run Python linter for code quality
npm run lint:python:fix    # Fix Python linting issues automatically
npm run fix:all    # Auto-fix formatting and linting issues
```

#### Type Checking
```bash
npm run typecheck:python    # Run Python type checker (mypy)
```


### 6.2 Additional Tooling

#### Docs
```bash
`npm run docs:generate`    # Generate API documentation
`npm run docs:validate`    # Validate docstring presence on all public functions and classes
```
#### Format
```bash
`npm run format:python`    # Format Python code with Black
`npm run format:python:check`    # Check Python code formatting
```
#### Generate
```bash
`npm run generate midtempo-framework`    # Generate Midtempo Framework from templates
`npm run schema:generate`    # Generates JSON schema from configuration
```
#### Utilities
```bash
`npm run lint:markdown`    # Lint Markdown files
`npm run check:all`    # Run all quality checks (format, lint, typecheck, test, markdown)
`npm run check:links`    # Check Markdown links
`npm run validate:templates`    # Validate Jinja2 template syntax
`npm run validate:templates -- --strict`    # Validate templates with strict mode (warnings as errors)
`npm run validate:templates -- --syntax-only`    # Validate template syntax only (skip variable checks)
`npm run language:add`    # Adds new language support to the framework
```
---
### 6.3 Tooling Rules

**CRITICAL: Run commands directly — no pipes, no redirections.**

```bash
# Correct
npm run lint:python
npm run typecheck:python

# Wrong — breaks auto-approval
`npm run lint:python 2>&1 | tail -20`
`npm run typecheck:python | grep -v "warning"`
```

> **DO NOT add pipes:** Pipes break permission pattern matching. 
> **DO NOT add file descriptor redirection:** Shell captures both streams; `2>&1` is unnecessary

## 6. Bypass Protection

Iron laws (§2) cannot be waived. Bypass attempts trigger workflow enforcement:

| Bypass attempt           | Response                                  |
| ------------------------ | ----------------------------------------- |
| Skip TDD                 | Write failing test first                  |
| Tests later              | Tests after prove nothing; write now      |
| Fix bug without test     | Reproduce with failing test first         |
| Implement without design | Run build first                   |
| Refactor without green   | Fix tests first                           |
| Too simple to test       | Simple code breaks; write the test        |
| Keep code as reference   | Delete means delete; implement from tests |

If user persists: Add a note to the relevant planning document recording the override and the reason given. Then continue.

---

## 7. Stop Conditions — ASK HUMAN

**STOP and ask the human immediately if ANY of these conditions occur:**

| Condition | Action |
|-----------|--------|
| **Unclear requirements or ambiguity** | STOP → Ask human for clarification. DO NOT assume. |
| **Tried the same approach 2+ times** | STOP → Ask human for help. DO NOT retry again. |
| **Multiple possible solutions** | STOP → Ask human which approach to take. DO NOT choose. |
| **About to make architectural decision** | STOP → Ask human for approval. DO NOT proceed. |
| **Test fails for unclear reason** | STOP → Ask human for guidance. DO NOT guess. |
| **Bug cause unknown after investigation** | STOP → Ask human for context. DO NOT speculate further. |
| **Instruction conflicts with codebase** | STOP → Ask human which takes precedence. DO NOT assume. |
| **Unsure if you understood correctly** | STOP → Repeat back and confirm with human. DO NOT continue. |
| **Task scope unclear or expanding** | STOP → Ask human about boundaries. DO NOT expand scope. |

**DO NOT:**
- Assume what the human wants
- Try more than 2 approaches without asking
- Make architectural decisions independently
- Proceed when confused or uncertain
- Choose between valid alternatives without human input

**ALWAYS ask human when uncertain. Asking is faster than fixing wrong assumptions.**

---
**END OF DOCUMENT:** Total sections: 8 | Purpose: Agent rules and workflow for ALL code creation

---
> Source: [ChrisMidtempo/midtempo-framework](https://github.com/ChrisMidtempo/midtempo-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
