## devflow-atlas

> DevFlow Atlas is a manual-first operating graph for indie software businesses.

# DevFlow Atlas — Claude Code Operating Guide

## Project identity
DevFlow Atlas is a manual-first operating graph for indie software businesses.

The core V1 loop is:
1. map product operations as a graph
2. attach routines directly to nodes
3. return later and see what needs attention
4. act from the graph, Today view, or Weekly review

## V1 product definition
Build DevFlow Atlas V1 as:
- authentication
- workspace creation
- one workspace with one or more products
- manual graph creation
- node and edge editing
- node notes
- node checklists
- node reminders
- basic runbook text on nodes
- Today view
- Weekly review
- onboarding template
- premium but restrained visual quality

## Non-goals for V1
Do not add:
- deep integrations
- repo/tool auto-discovery
- AI-generated suggestions
- real-time collaboration
- advanced analytics
- complex billing logic
- real-time multiplayer editing

## Locked stack
The stack is locked in `docs/architecture/STACK_LOCK.md`.

Do not change:
- framework
- router
- auth provider
- database strategy
- graph library
- ORM
- deployment target
unless the user explicitly approves a stack change.

## Read order at session start
1. `COMPACT_CONTEXT.md`
2. `PHASE_HANDOFF.md`
3. `PROJECT_MEMORY.md`
4. `PHASE_STATE.json`
5. `CLAUDE.md`
6. `docs/architecture/STACK_LOCK.md`
7. `docs/plan/IMPLEMENTATION_PHASES.md`
8. `docs/plan/IMPLEMENTATION_SLICES.md`
9. `docs/product/PRD-v1.md` only if the current work genuinely needs it

## Phase gate
- The current allowed phase is defined by `PHASE_STATE.json`.
- Only create or update files required by the current phase or current slice.
- Never begin the next phase unless:
  1. the user explicitly approves it, and
  2. `PHASE_STATE.json` has been manually advanced.
- Never edit `PHASE_STATE.json` unless the user explicitly instructs phase advancement.

## Execution rules
1. Work only inside the current phase and slice.
2. Prefer the smallest correct implementation.
3. Do not create speculative abstractions.
4. Prefer updating an existing file over creating a near-duplicate.
5. Do not create extra docs beyond the current phase requirements.
6. Run preflight before implementation work.
7. Run verification after implementation work.
8. Stop immediately after the current slice is complete.

## Failure protocol
When something fails:
1. Stop broad changes immediately.
2. Run `./scripts/phase-verify.sh`.
3. Inspect the first failing command only.
4. If a log file exists, use:
   - `./scripts/cc-first-error.sh <logfile> 25`
   - `./scripts/cc-tail-failures.sh <logfile> 120`
5. Fix the smallest failing scope.
6. Do not continue to the next slice until verification passes.
7. Do not change architecture to fix a local implementation bug.
8. Ask the user only if blocked by:
   - missing credentials
   - unavailable external service
   - an explicit stack change decision

## Documentation roles
- `PROJECT_MEMORY.md` stores durable truths and locked decisions.
- `COMPACT_CONTEXT.md` stores the current phase, objective, and immediate constraints.
- `PHASE_HANDOFF.md` stores the exact continuation instructions for the next session.
- `PHASE_STATE.json` stores the manually controlled phase lock.
- `docs/architecture/STACK_LOCK.md` stores the fixed technical stack.
- `docs/plan/IMPLEMENTATION_PHASES.md` stores the roadmap.
- `docs/plan/IMPLEMENTATION_SLICES.md` stores execution-grade slices.

## Important rule
Marketing copy is not required engineering context.
Keep homepage copy and positioning out of normal coding session reads unless the task is explicitly marketing-related.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MerverliPy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MerverliPy)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
