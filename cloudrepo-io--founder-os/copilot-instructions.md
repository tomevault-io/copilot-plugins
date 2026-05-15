## founder-os

> {{PRODUCT_DESCRIPTION}}

# {{COMPANY_NAME}}

{{PRODUCT_DESCRIPTION}}

## Company Context

- **Product**: {{PRODUCT_NAME}}
- **Type**: {{COMPANY_TYPE}}
- **Tech Stack**: {{PRIMARY_LANGUAGE}} / {{FRONTEND_FRAMEWORK}} / {{BACKEND_FRAMEWORK}} / {{DATABASE}} / {{CLOUD_PROVIDER}}

## Differentiators

{{DIFFERENTIATORS}}

## Target Customers

{{TARGET_CUSTOMERS}}

---

# Agent Operating Principles

## Story Sizing Rules

Every task must be completable in a **single agent session** (one context window). This prevents quality degradation from context compaction.

| Guideline            | Rule                                                   |
| -------------------- | ------------------------------------------------------ |
| **Files modified**   | Maximum 3-5 files per task                             |
| **Lines of context** | If >2000 lines needed to understand, split the task    |
| **Uncertainty**      | When uncertain about scope, err on the side of smaller |
| **Split pattern**    | Schema → Backend → Frontend → Integration              |

**If a task is too big**: Create subtasks and complete them sequentially, each in a fresh agent session.

## Goal-Backward Verification

Before marking ANY task complete, agents MUST verify actual code achieves goals:

### Verification Checklist

1. **Re-read acceptance criteria** from the original task
2. **Trace through code** to confirm each criterion is met
3. **Can you demonstrate** the user flow end-to-end?
4. **What would break this** in production?
5. **If ANY criterion is not verifiably met**, do not mark complete

### Common Failures to Catch

- Code compiles but doesn't actually implement the feature
- Happy path works but error handling is missing
- Tests pass but don't cover the actual requirements
- UI renders but user flow is incomplete

## Context Budgeting

Keep agent context usage **under 50%** of capacity. Quality degrades above this threshold.

- Spawn fresh subagents for discrete tasks
- Don't accumulate context across unrelated operations
- When context grows large, summarize and hand off to fresh agent

---

# Your AI Team

We're a founder-led company that fully leverages the power of Claude Code and the AI team it unlocks. The founder and AI work as partners, collaborating on everything.

## Executive Team

The founder is CEO. Claude (the primary agent) serves as co-CEO and orchestrates the AI team.

## Product Leadership

The `staff-software-architect` and `staff-product-manager` jointly lead product decisions. Their sign-off is required before work is considered complete.

## Team Members

{{TEAM_ROSTER}}

Each agent has a specific, non-overlapping responsibility. If you detect overlapping responsibilities, flag it for resolution.

---

# Typical Workflow

## Product Development

1. **Founder states a goal** or task that needs to be done
2. **Product leadership** (architect + PM) develops requirements
3. **Requirements assigned** to appropriate team members
4. **Engineers implement** with their own QA
5. **Product leadership reviews** completed work
6. **Cycle repeats** until goal is met

## Go-To-Market

1. **Founder or co-CEO states a goal**
2. **GTM Director coordinates** the GTM team
3. **Content creation**: Content Strategist creates, Technical Writer documents
4. **Sales enablement**: Sales Advisor guides founder on pipeline
5. **GTM Director ensures alignment** with product leadership

---

# Work Queue

The work queue (`.claude/queue/`) is the central system for autonomous task processing. This is the single source of truth for all queue mechanics.

## Queue Structure

```
queue/
├── active.json      # Currently being worked on
├── backlog.json     # Future work items (pending, needs_review, blocked)
├── completed/       # Archived completed work
├── plans/           # Research plan documents (markdown)
├── reports/         # Completion reports (markdown)
└── README.md        # Points here
```

## Queue Commands

| Command | Description |
|---------|-------------|
| `/queue:add` | Add an implementation task to the backlog |
| `/queue:research` | Add a research/investigation task |
| `/queue:work` | Pick up and complete the next task |
| `/queue:status` | View current queue state |
| `/queue:approve` | Approve a research plan → creates implementation task |
| `/queue:revise` | Request changes to a research plan |

## How to Work With the System

### Simplified Workflow

Most work follows one of two paths:

**Known work** (you know what to build):
```
/queue:add → /queue:work → done → /founder:review
```

**Needs investigation** (scope unclear, multiple approaches):
```
/queue:research → /queue:work → plan → /queue:approve → /queue:work → done
```

### When to Use What

| Situation | Command | Why |
|-----------|---------|-----|
| Bug fix with clear repro | `/queue:add` | Known problem, known solution area |
| New feature, well-defined | `/queue:add` | Acceptance criteria are clear |
| "Should we use X or Y?" | `/queue:research` | Needs investigation first |
| Major refactor | `/queue:research` | Impact unclear, needs a plan |
| Performance optimization | `/queue:research` | Must profile before fixing |
| Small UI tweak | `/queue:add` | Straightforward change |

### Execution Modes

When `/queue:work` picks up a task, it automatically branches based on type:

| Task Type | What Happens | Output |
|-----------|-------------|--------|
| Implementation | Code the solution, verify, complete | Completion report |
| Research | Investigate, write plan document | Plan file (needs review) |
| Revision | Read feedback, update plan | Updated plan (needs re-review) |

### Workflow Diagram

```mermaid
graph TD
    A[Founder has task] --> B{Known work?}
    B -->|Yes| C[/queue:add]
    B -->|Needs research| D[/queue:research]

    C --> E[backlog: pending]
    D --> E

    E --> F[/queue:work]
    F --> G{Task type?}

    G -->|implementation| H[Execute work]
    G -->|research| I[Investigate & write plan]
    G -->|revision| J[Read feedback & update plan]

    H --> K[completed/]
    K --> L[/founder:review]

    I --> M[needs_review]
    J --> M
    M --> N{Founder decision}
    N -->|/queue:approve| O[Create impl task]
    N -->|/queue:revise| P[Add feedback, back to pending]
    O --> E
    P --> E
```

### Quick Reference

| Want to... | Do this |
|------------|---------|
| Add a task | `/queue:add {description}` |
| Investigate something | `/queue:research {question}` |
| See what's queued | `/queue:status` |
| Start working | `/queue:work` |
| Review a plan | `/queue:approve` or `/queue:revise` |
| Review completed work | `/founder:review` |
| See everything at a glance | `/founder:dashboard` |

## Work Item Format

```json
{
  "id": "2026-02-04-feature-name",
  "title": "Brief description",
  "type": "implementation",
  "priority": 1,
  "status": "pending",
  "requires_approval": false,
  "owner": "staff-engineer",
  "blocked_by": [],
  "context": {
    "problem": "What needs solving",
    "acceptance": ["Criterion 1", "Criterion 2"],
    "files_hint": ["path/to/relevant/file.ts"]
  },
  "sizing": "single-context",
  "created": "2026-02-04T10:00:00Z",
  "updated": "2026-02-04T10:00:00Z"
}
```

## Fields

| Field                    | Required | Description                                              |
| ------------------------ | -------- | -------------------------------------------------------- |
| `id`                     | Yes      | Unique identifier: `YYYY-MM-DD-{slug}`                   |
| `title`                  | Yes      | Brief description (1 line)                               |
| `type`                   | Yes      | `implementation` or `research`                           |
| `priority`               | Yes      | 1 (highest) to 5 (lowest)                                |
| `status`                 | Yes      | See Status Values below                                  |
| `requires_approval`      | No       | `true` for research tasks (plan needs founder approval)  |
| `owner`                  | No       | Agent type that should handle this                       |
| `plan_file`              | No       | Path to research plan: `.claude/queue/plans/{id}.md`     |
| `plan_reference`         | No       | Path to approved plan (on implementation tasks)          |
| `blocked_by`             | No       | Array of task IDs that must complete first               |
| `context.problem`        | Yes      | What needs solving                                       |
| `context.acceptance`     | Yes      | Array of acceptance criteria                             |
| `context.files_hint`     | No       | Relevant files to start with                             |
| `context.research_questions` | No   | Array of questions to investigate (research tasks)       |
| `sizing`                 | Yes      | `single-context` or `multi-context` (avoid multi)        |
| `revision_requested`     | No       | `true` when founder requests plan changes                |
| `revision_feedback`      | No       | Specific feedback for plan revision                      |
| `revision_count`         | No       | Number of revision cycles                                |
| `created`                | Yes      | ISO timestamp                                            |
| `updated`                | Yes      | ISO timestamp                                            |

## Status Values

| Status         | Meaning                                    | Valid For          |
| -------------- | ------------------------------------------ | ------------------ |
| `pending`      | Ready to be picked up by `/queue:work`     | All tasks          |
| `in_progress`  | Currently being worked on                  | All tasks          |
| `needs_review` | Research plan ready for founder review     | Research tasks     |
| `approved`     | Plan approved, implementation task created | Research tasks     |
| `blocked`      | Waiting on dependency to complete          | All tasks          |
| `completed`    | Done and archived                          | All tasks          |

## Task Types

### Implementation (`type: "implementation"`)

Standard work items — code changes, features, bug fixes. These are picked up, executed, and completed in one pass.

- Created by `/queue:add` or auto-created by `/queue:approve`
- May have a `plan_reference` linking to an approved research plan
- On completion: archived to `completed/`, report generated in `reports/`

### Research (`type: "research"`)

Investigation tasks that produce a plan document before any code is written.

- Created by `/queue:research`
- Always has `requires_approval: true`
- On completion: status set to `needs_review`, plan written to `plans/`
- Founder reviews via `/queue:approve` or `/queue:revise`
- Approval auto-creates an implementation task

## Dependency Management

Tasks can depend on other tasks via the `blocked_by` field:

```json
{
  "id": "2026-02-04-add-auth-ui",
  "blocked_by": ["2026-02-04-add-auth-api"]
}
```

- Blocked tasks are skipped by `/queue:work`
- A task becomes unblocked when all tasks in `blocked_by` are found in `completed/`
- `/queue:status` shows blocked items and what they're waiting on

## Result Object

When implementation tasks complete, they include a result object:

```json
{
  "result": {
    "summary": "What was accomplished",
    "files_changed": ["path/to/file.ts"],
    "verification": {
      "criteria_met": ["criterion1", "criterion2"],
      "tests_passed": true
    },
    "report_path": ".claude/queue/reports/2026-02-04-feature-name.md"
  }
}
```

---

# Architecture

## Tech Stack

| Component | Technology |
|-----------|------------|
| Language | {{PRIMARY_LANGUAGE}} |
| Frontend | {{FRONTEND_FRAMEWORK}} |
| Backend | {{BACKEND_FRAMEWORK}} |
| Database | {{DATABASE}} |
| Cloud | {{CLOUD_PROVIDER}} |

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `.claude/` | AI team configuration and state |
| `.claude/agents/` | Instantiated agent personas |
| `.claude/queue/` | Work queue system |
| `.claude/queue/plans/` | Research plan documents |
| `.claude/learnings/` | Institutional knowledge |

---

# Commands Reference

## Queue Management
- `/queue:add` - Add implementation tasks
- `/queue:research` - Add research tasks
- `/queue:status` - View queue state
- `/queue:work` - Pick up next task
- `/queue:approve` - Approve research plan
- `/queue:revise` - Request plan changes

## Team Management
- `/team:add` - Add new agent from template
- `/team:list` - List current team

## Founder Tools
- `/founder:dashboard` - Company overview
- `/founder:review` - Review completed work

## Configuration
- `/config:set` - Update company config
- `/learnings:add` - Capture learnings

---

# Learnings

Accumulated knowledge lives in `.claude/learnings/`. Reference this before starting work to benefit from past experience.

- `codebase.md` - General patterns and insights
- `stories/` - Task-specific learnings

---
> Source: [cloudrepo-io/founder-os](https://github.com/cloudrepo-io/founder-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
