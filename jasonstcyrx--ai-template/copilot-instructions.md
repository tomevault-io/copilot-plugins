## ai-template

> - Tickets must follow this progression:

# Ticket Workflow Rules

## States and Transitions
- Tickets must follow this progression:
  1. `backlog` → Initial state for all new tickets
  2. `todo` → Ready for development
  3. `in-progress` → Currently being worked on
  4. `review` → Ready for code review
  5. `done` → Completed and verified
  - Special states:
    - `blocked` → Blocked by dependencies
    - `archive` → Historical record

## State Rules
- Only move tickets to states defined in their `next_states`
- Update `updated_at` timestamp on every state change
- Add a comment explaining why when moving to `blocked`
- Archive tickets after 30 days in `done`

## Required Fields
Each ticket type has mandatory fields:

### Bug
- Steps to reproduce
- Expected behavior
- Actual behavior
- Severity (critical, high, medium, low)

### Feature
- User story
- Acceptance criteria
- Business value

### Task
- Description
- Deliverables

### Spike
- Questions to answer
- Time box
- Expected outcomes

### ONCE YOU ARE DONE WITH WORK IF NEW TICKETS NEED TO BE CREATED THEN CREATE THEM.

You must complete the documentation for your work and all of the tests. e2e, unit and api testing all should pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasonstcyrx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
