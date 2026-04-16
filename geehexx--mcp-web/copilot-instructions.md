## mcp-web

> Apply when routing work making workflow decisions or detecting project context


# Workflow Routing and Signal Detection

**Purpose:** Quick reference for `/work` routing decisions based on detected signals.

**Authority:** [work-routing.md](../workflows/work-routing.md)

---

## Confidence Levels

| Confidence | Threshold | Action | User Confirmation |
|------------|-----------|--------|-------------------|
| **High** | 80%+ | Auto-proceed | Not needed |
| **Medium** | 30-79% | Auto-proceed with alternatives | State recommendation, mention alternatives |
| **Low** | <30% | Prompt user | Ask for direction |

---

## Signal Priority Matrix

| Signal | Confidence | Route To | Rationale |
|--------|-----------|----------|-----------|
| **Active initiative with "In Progress" tasks** | High (90%) | `/implement` | Clear continuation point |
| **Test failures detected** | High (85%) | Fix immediately or `/implement` | Blocking issue |
| **Uncommitted changes** | Medium (70%) | `/commit` | Clean state needed |
| **Initiative marked "Planning"** | High (80%) | `/plan` | Explicit planning phase |
| **Initiative marked "Completed"** | High (95%) | `/archive-initiative` | Cleanup required |
| **Multiple initiatives active** | Medium (50%) | Prompt user | Ambiguous priority |
| **Recent session summary exists** | Medium (60%) | Continue from summary | Context available |
| **Clean slate (no signals)** | Low (20%) | Prompt user | No clear direction |

---

## Common Routing Scenarios

### Scenario 1: Active Initiative

**Signals:**

- Initiative file exists in `docs/initiatives/active/`
- Status: "Active" or "In Progress"
- Unchecked tasks present

**Confidence:** High (90%)

**Route:** `/implement`

**Reasoning:** Clear continuation point with defined tasks

---

### Scenario 2: Test Failures

**Signals:**

- `task test:fast` returns non-zero exit code
- Test output shows failures

**Confidence:** High (85%)

**Route:** Fix immediately or `/implement`

**Reasoning:** Blocking issue that prevents further work

---

### Scenario 3: Planning Markers

**Signals:**

- Initiative status: "Planning"
- No implementation tasks yet
- Research needed

**Confidence:** High (80%)

**Route:** `/plan`

**Reasoning:** Explicit planning phase indicated

---

### Scenario 4: Completed Initiative

**Signals:**

- Initiative status: "Completed" or "✅"
- All tasks checked
- No uncommitted changes

**Confidence:** High (95%)

**Route:** `/archive-initiative`

**Reasoning:** Cleanup required before new work

---

### Scenario 5: Uncommitted Changes

**Signals:**

- `git status --short` shows modified files
- No active initiative

**Confidence:** Medium (70%)

**Route:** `/commit`

**Reasoning:** Clean state needed, but unclear if work is complete

---

### Scenario 6: Multiple Active Initiatives

**Signals:**

- 2+ initiatives in `docs/initiatives/active/`
- No clear priority indicators

**Confidence:** Medium (50%)

**Route:** Prompt user

**Reasoning:** Ambiguous priority, user input needed

---

### Scenario 7: Recent Session Summary

**Signals:**

- Session summary exists in `docs/archive/session-summaries/`
- Date within last 7 days
- Summary indicates next steps

**Confidence:** Medium (60%)

**Route:** Continue from summary context

**Reasoning:** Context available, but may be stale

---

### Scenario 8: Clean Slate

**Signals:**

- No active initiatives
- No uncommitted changes
- No test failures
- No recent session summaries

**Confidence:** Low (20%)

**Route:** Prompt user

**Reasoning:** No clear direction, user input required

---

## Decision Tree

```text
┌─────────────────────────────────────┐
│ /detect-context returns signals     │
└──────────────┬──────────────────────┘
               │
               ▼
       ┌───────────────┐
       │ Test failures?│
       └───────┬───────┘
               │
         Yes ──┼── No
               │       │
               ▼       ▼
         Fix tests  ┌──────────────────┐
                    │ Completed init?  │
                    └────────┬─────────┘
                             │
                       Yes ──┼── No
                             │       │
                             ▼       ▼
                    /archive-init  ┌────────────────┐
                                   │ Active init?   │
                                   └────────┬───────┘
                                            │
                                      Yes ──┼── No
                                            │       │
                                            ▼       ▼
                                      /implement  ┌──────────────┐
                                                  │ Planning?    │
                                                  └──────┬───────┘
                                                         │
                                                   Yes ──┼── No
                                                         │       │
                                                         ▼       ▼
                                                     /plan   Prompt user
```

---

## Routing Examples

### Example 1: High Confidence

**Detected signals:**

```yaml
active_initiatives:
  - path: docs/initiatives/active/2025-10-20-feature-x/initiative.md
    status: Active
    progress: 60%
    unchecked_tasks: 5

git_status: clean
test_status: passing
```

**Routing decision:**

```text
Confidence: 90% (High)
Route: /implement
Rationale: Active initiative with clear tasks
Action: Auto-proceed without user confirmation
```

---

### Example 2: Medium Confidence

**Detected signals:**

```yaml
active_initiatives:
  - path: docs/initiatives/active/2025-10-18-refactor-y/initiative.md
    status: Active
    progress: 30%
  - path: docs/initiatives/active/2025-10-19-feature-z/initiative.md
    status: Active
    progress: 50%

git_status: clean
test_status: passing
```

**Routing decision:**

```text
Confidence: 50% (Medium)
Route: Prompt user
Rationale: Multiple active initiatives, unclear priority
Action: Present options:
  1. Continue feature-z (50% complete, more recent)
  2. Continue refactor-y (30% complete)
  3. Start new work
```

---

### Example 3: Low Confidence

**Detected signals:**

```yaml
active_initiatives: []
git_status: clean
test_status: passing
recent_summaries: []
```

**Routing decision:**

```text
Confidence: 20% (Low)
Route: Prompt user
Rationale: No clear signals, clean slate
Action: Ask: "What would you like to work on?"
```

---

## Override Signals

**User can override routing with explicit commands:**

| User Input | Override Route | Confidence |
|------------|---------------|------------|
| "Continue initiative X" | `/implement` (X) | 100% |
| "Plan new feature Y" | `/plan` | 100% |
| "Run tests" | `/validate` | 100% |
| "Commit changes" | `/commit` | 100% |
| "Archive completed work" | `/archive-initiative` | 100% |

---

## References

- [work-routing.md](../workflows/work-routing.md) - Complete routing logic
- [detect-context.md](../workflows/detect-context.md) - Signal detection
- [work.md](../workflows/work.md) - Main orchestrator workflow

---

**Maintained by:** mcp-web core team
**Version:** 1.0.0

---

## Rule Metadata

**File:** `13_workflow_routing.md`
**Trigger:** model_decision
**Estimated Tokens:** ~1,800
**Last Updated:** 2025-10-21
**Status:** Active

**Can be @mentioned:** Yes (hybrid loading)

**Topics Covered:**

- Routing matrix
- Signal detection
- Confidence levels
- Decision tree

**Workflow References:**

- /work - Work routing
- /work-routing - Routing decisions

**Dependencies:**

- Source: workflow-routing-matrix.md

**Changelog:**

- 2025-10-21: Created from workflow-routing-matrix.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/geehexx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
