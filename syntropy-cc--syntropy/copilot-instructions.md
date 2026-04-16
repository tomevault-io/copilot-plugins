## syntropy

> Rules for tracking implementation progress and metrics


# Progress Tracking Rules

## Purpose

This file defines how implementation progress is tracked, measured, and reported. Accurate progress tracking enables informed decision-making, identifies blockers early, and maintains a clear picture of project health. The tracking system integrates tightly with architecture to ensure nothing is forgotten.

> **AI Instruction**: Update progress tracking documents whenever you complete work, encounter blockers, or discover new information. Accurate tracking is as important as the implementation itself.

## Work Item Structure

### Rule TRACK-001: Work Item Identifier Format

Every work item has a unique identifier following this pattern:

```
{TYPE}-{SEQUENCE}[.{SUB}]

Types:
- COMP: Component implementation (from architecture)
- FEAT: Feature implementation
- TASK: Technical task
- FIX:  Bug fix
- DEBT: Technical debt remediation
- DOC:  Documentation task

Examples:
- COMP-001     : First component implementation
- COMP-001.1   : Sub-task of COMP-001
- COMP-001.2   : Another sub-task
- FEAT-015     : Feature implementation
- TASK-042     : Technical task
- FIX-003      : Bug fix
- DEBT-007     : Technical debt item
```

### Rule TRACK-002: Work Item Required Fields

Every work item MUST have these fields:

```markdown
### [{ID}] {Title}

| Field | Value |
|-------|-------|
| **Status** | Backlog / Ready / In Progress / Review / Done |
| **Priority** | Critical / High / Medium / Low |
| **Origin** | {Architecture reference: ARCH-XXX, ADR-XXX, or RFC-XXX} |
| **Component** | {Component name from architecture} |
| **Dependencies** | {List of item IDs that must be Done first} |
| **Estimated Size** | XS / S / M / L / XL |
| **Created** | {YYYY-MM-DD} |
| **Updated** | {YYYY-MM-DD} |

{Brief description of what needs to be implemented}

#### Acceptance Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] {Criterion 3}
```

### Rule TRACK-003: Size Estimation Guidelines

Size represents relative effort, not time:

| Size | Description | Typical Scope |
|------|-------------|---------------|
| XS | Trivial change | Single function, < 20 lines |
| S | Small task | Single file, clear solution |
| M | Medium task | Multiple files, one concept |
| L | Large task | Multiple concepts, needs planning |
| XL | Very large | Should be broken down further |

> **AI Instruction**: If an item is estimated as XL, break it into smaller items before starting. No item should require more than 3-5 prompts to complete.

## Status Management

### Rule TRACK-004: Status Definitions and Transitions

```
┌──────────────────────────────────────────────────────────────────┐
│                      Status State Machine                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌─────────┐     Prioritized      ┌─────────┐                   │
│   │ BACKLOG │─────────────────────▶│  READY  │                   │
│   └─────────┘                      └────┬────┘                   │
│        ▲                                │                         │
│        │                           Work Started                   │
│        │                                │                         │
│        │         ┌──────────────────────▼─────────────────────┐  │
│        │         │              IN PROGRESS                    │  │
│        │         │  ┌─────────────────────────────────────┐   │  │
│        │         │  │ Sub-states:                         │   │  │
│   Blocked/       │  │ • Implementing (default)            │   │  │
│   Needs Rework   │  │ • Blocked (waiting on dependency)   │   │  │
│        │         │  │ • Paused (context switch)           │   │  │
│        │         │  └─────────────────────────────────────┘   │  │
│        │         └──────────────────────┬─────────────────────┘  │
│        │                                │                         │
│        │                           Code Complete                  │
│        │                                │                         │
│        │                          ┌─────▼─────┐                   │
│        └──────────────────────────│  REVIEW   │                   │
│                                   └─────┬─────┘                   │
│                                         │                         │
│                                    All Checks Pass                │
│                                         │                         │
│                                   ┌─────▼─────┐                   │
│                                   │   DONE    │                   │
│                                   └───────────┘                   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Transition Rules**:

| From | To | Trigger | Required Action |
|------|-----|---------|-----------------|
| Backlog | Ready | Prioritization | Move to CURRENT-WORK.md |
| Ready | In Progress | Start work | Update component record |
| In Progress | Blocked | Dependency found | Document blocker |
| In Progress | Review | Code complete | Run completion checklist |
| Review | Done | All verified | Update metrics |
| Review | In Progress | Issues found | Document issues |
| Any | Backlog | Deprioritized | Document reason |

### Rule TRACK-005: Blocked Item Handling

When an item becomes blocked:

```markdown
### [{ID}] {Title}
| **Status** | In Progress (Blocked) |
| **Blocked By** | {ITEM-ID or external reason} |
| **Blocked Since** | {YYYY-MM-DD} |
| **Unblock Action** | {What needs to happen to unblock} |
```

**Blocker Categories**:
- **Dependency**: Waiting on another work item
- **External**: Waiting on external team/service
- **Technical**: Needs investigation or decision
- **Resource**: Waiting on access/credentials/infrastructure

> **AI Instruction**: When you encounter a blocker, immediately update the item status, document the blocker, and suggest alternative work that can proceed.

## Progress Metrics

### Rule TRACK-006: Required Metrics

Track these metrics in PROGRESS-SUMMARY.md:

#### Velocity Metrics

```markdown
## Velocity Metrics

### Current Period ({start_date} - {end_date})

| Metric | Value | Trend |
|--------|-------|-------|
| Items Started | {N} | ↑/↓/→ |
| Items Completed | {N} | ↑/↓/→ |
| Items Blocked | {N} | ↑/↓/→ |
| Completion Rate | {completed/started}% | ↑/↓/→ |

### Historical (Last 4 Periods)

| Period | Started | Completed | Blocked | Rate |
|--------|---------|-----------|---------|------|
| Current | N | N | N | N% |
| Previous | N | N | N | N% |
| ... | ... | ... | ... | ... |
```

#### Architecture Coverage Metrics

```markdown
## Architecture Coverage

### Component Implementation Status

| Component | Status | Items | Done | Coverage |
|-----------|--------|-------|------|----------|
| {Component 1} | Complete | 5 | 5 | 100% |
| {Component 2} | In Progress | 8 | 3 | 37.5% |
| {Component 3} | Not Started | 4 | 0 | 0% |
| **Total** | | **17** | **8** | **47%** |

### Layer Coverage

| Layer | Components | Implemented | Coverage |
|-------|------------|-------------|----------|
| Domain | N | N | N% |
| Application | N | N | N% |
| Infrastructure | N | N | N% |
| Presentation | N | N | N% |
```

#### Quality Metrics

```markdown
## Quality Metrics

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Test Coverage | 80% | {N}% | ✅/⚠️/❌ |
| Items with Tests | 100% | {N}% | ✅/⚠️/❌ |
| Documentation Coverage | 100% | {N}% | ✅/⚠️/❌ |
| Technical Debt Items | < 10 | {N} | ✅/⚠️/❌ |
```

### Rule TRACK-007: Metric Update Frequency

| Metric Category | Update Trigger |
|-----------------|----------------|
| Item counts | On any status change |
| Completion rate | On item completion |
| Architecture coverage | On component completion |
| Quality metrics | Weekly or on milestone |
| Velocity trends | End of each period |

### Rule TRACK-008: Architecture Coverage Calculation

```
Component Coverage = (Done items for component / Total items for component) × 100

Layer Coverage = (Implemented components in layer / Total components in layer) × 100

Overall Coverage = (Total Done items / Total items) × 100
```

A component is considered "Implemented" when:
- All its work items are Done
- Tests are passing (verified)
- Documentation exists (verified)

## Component Record Management

### Rule TRACK-009: Component Record Lifecycle

Every architectural component gets a dedicated implementation record:

```
1. Architecture defines component
   └─▶ Create component record file
       └─▶ Extract work items to BACKLOG.md
           └─▶ Implement items (update record during work)
               └─▶ Mark component complete
                   └─▶ Record evolves with architecture changes
```

**File Naming**: `{COMPONENT-ID}-{component-name}.md`

Examples:
- `COMP-001-user-authentication.md`
- `COMP-002-order-processing.md`
- `COMP-003-notification-service.md`

### Rule TRACK-010: Component Record Required Sections

```markdown
# {Component Name} Implementation Record

## Metadata
| Field | Value |
|-------|-------|
| **Component ID** | COMP-XXX |
| **Architecture Ref** | [Component Spec](link) |
| **Status** | Not Started / In Progress / Complete |
| **Owner** | {Person/Team} |
| **Created** | {YYYY-MM-DD} |
| **Last Updated** | {YYYY-MM-DD} |

## Architecture Summary
Brief description of what this component does, copied/linked from architecture.

## Work Items
| ID | Title | Status | Size |
|----|-------|--------|------|
| COMP-XXX.1 | {Task 1} | Done | S |
| COMP-XXX.2 | {Task 2} | In Progress | M |
| COMP-XXX.3 | {Task 3} | Ready | S |

## Implementation Decisions
Decisions made during implementation that aren't in ADRs.

## Dependencies
### Requires (This depends on)
- {Component/Service}: {Why}

### Required By (Depends on this)
- {Component/Service}: {Why}

## Implementation Log
Chronological log of significant events.

## Lessons Learned
What worked well, what could be improved.

## Evolution History
Record of changes when architecture evolves.
```

## Integration with Architecture

### Rule TRACK-011: Architecture Sync Process

When architecture documents change:

```markdown
## Architecture Sync Checklist

When ARCHITECTURE.md or ADRs change:

1. [ ] Identify new/modified components
2. [ ] For each affected component:
   - [ ] Create/update component record
   - [ ] Create new work items in BACKLOG.md
   - [ ] Mark existing items for update if needed
3. [ ] Update PROGRESS-SUMMARY.md component list
4. [ ] Update dependency graph if relationships changed
5. [ ] Prioritize new items in CURRENT-WORK.md
```

### Rule TRACK-012: Traceability Requirements

Every work item must be traceable:

```
Architecture Document
       │
       │ defines
       ▼
Component Specification ─────────────▶ Component Record
       │                                     │
       │ generates                           │ contains
       ▼                                     ▼
Work Items (BACKLOG.md) ◀──────────── Implementation Details
       │                                     │
       │ implemented as                      │ documented in
       ▼                                     ▼
Source Code ◀───────────────────────── Implementation Log
```

**Traceability Markers**:
- In code: `# Architecture: COMP-XXX, ADR-YYY`
- In tests: `# Tests for: COMP-XXX`
- In docs: `Architecture reference: [COMP-XXX](link)`

## Reporting

### Rule TRACK-013: Progress Summary Structure

PROGRESS-SUMMARY.md provides executive overview:

```markdown
# Implementation Progress Summary

> Last Updated: {YYYY-MM-DD HH:MM}

## Executive Summary

{2-3 sentences on current state: what's progressing, what's blocked, key achievements}

## Key Metrics

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Overall Progress | N% | 100% | 🟡 |
| Items Complete | N/M | M | 🟡 |
| Items Blocked | N | 0 | 🔴 |
| Test Coverage | N% | 80% | 🟢 |

## Current Focus

Top 3-5 items currently being worked on.

## Recent Completions

Items completed in last period.

## Blockers & Risks

Active blockers and mitigation status.

## Upcoming Milestones

Next significant deliverables.
```

### Rule TRACK-014: Status Indicators

Use consistent visual indicators:

| Indicator | Meaning |
|-----------|---------|
| 🟢 | On track / Good |
| 🟡 | At risk / Needs attention |
| 🔴 | Blocked / Critical |
| ⬜ | Not started |
| 🔵 | In progress |
| ✅ | Complete |
| ⏸️ | Paused |
| 🚫 | Cancelled |

## Automation Hints

### Rule TRACK-015: Parseable Format

Use consistent markdown structure that could be parsed:

```markdown
### [COMP-001.1] Create User entity
| **Status** | Done |
| **Completed** | 2024-01-15 |

### [COMP-001.2] Implement password hashing
| **Status** | In Progress |
| **Started** | 2024-01-16 |
```

This enables future automation to extract metrics automatically.

### Rule TRACK-016: Update Commands

When updating documents, use clear patterns:

```markdown
<!-- For AI: Update status -->
Change status of COMP-001.2 from "In Progress" to "Done"
Set completion date to today

<!-- For AI: Add new item -->
Add new work item:
- ID: COMP-001.5
- Title: Add email validation
- Origin: ADR-023
- Size: S
- Status: Backlog

<!-- For AI: Update metrics -->
Recalculate:
- Total items count
- Done items count  
- Coverage percentage
```

> **AI Instruction**: When you update tracking documents, explicitly state what you're changing. Use the patterns above so changes are clear and auditable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syntropy-cc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/syntropy-cc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
