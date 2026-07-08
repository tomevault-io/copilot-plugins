## aieos-business-process-kit

> This is the **Business Process Kit** — an AIEOS kit that governs the organizational and process change side of solution delivery. It is Layer 15 of the AIEOS system, operating as a cross-cutting governance kit that ensures business processes affected by technical changes are identified, planned for, and confirmed ready before release.

# CLAUDE.md — Business Process Kit

## What This Repository Is

This is the **Business Process Kit** — an AIEOS kit that governs the organizational and process change side of solution delivery. It is Layer 15 of the AIEOS system, operating as a cross-cutting governance kit that ensures business processes affected by technical changes are identified, planned for, and confirmed ready before release.

BPK addresses the gap where technically sound solutions ship but no thought is given to how business processes are impacted — leading to adoption failures, broken workflows, and untrained teams.

## Repository Structure

```
docs/
  principles/          # Organizational process change policy (input material)
  specs/               # Content rules and hard gates per artifact type
  artifacts/           # Templates
  prompts/             # AI generation prompts
  validators/          # Quality gate definitions
  playbook.md          # End-to-end process definition
  index.md             # Documentation entry point
  how-to-adapt.md      # Organizational adoption guidance
  how-to-use-with-ai.md # AI tool usage guide
  governance-model.md  # AIEOS structural rules (reference)
  entry-from-eek.md    # Boundary briefing from Engineering Execution Kit
examples/              # Worked examples
tests/                 # Structural integrity checks
```

## Artifact Types

This kit produces three governed artifact types:

- **Process Impact Assessment (PIA)** — Identifies every business process affected by the initiative, classifies the impact, and maps affected roles. 5 hard gates.
- **Transition Plan (TP)** — Defines how affected users move from current-state to future-state processes: transition strategy, communication, training, cutover schedule, rollback procedures. 5 hard gates.
- **Readiness Confirmation (RC)** — Evidence that affected teams are actually prepared: training completion, SOP updates, stakeholder acknowledgments. 4 hard gates.

Each artifact type has exactly four governing files: spec, template, prompt, validator.

## Key Rules

- **Specs are the source of truth** — prompts and validators reference specs, never inline rules
- **Validators judge, they do not help** — no suggestions, no redesign
- **Freeze-before-promote** — PIA frozen before TP; TP frozen before RC
- **Separate generation and validation** — different AI sessions to prevent self-validation bias
- **Evidence-based readiness** — RC captures what has been done, not what is planned
- **Upstream traceability** — every process impact must trace to a SAD component or TDD decision
- **Governance model sync** — `docs/governance-model.md` is a synchronized copy of `aieos-governance-foundation/governance-model.md` (canonical authority). Do not edit kit copy directly; update `aieos-governance-foundation` first, then sync all kit copies to match exactly.
- **Engagement Record** — BPK maintains the Layer 15 section of the project's ER. See `docs/playbook.md §Maintaining the Engagement Record`.

## Artifact Flow

```
Trigger: SAD or TDD frozen (EEK)
         │
Step 1: Process Impact Assessment (PIA) — generated
         │ Validate → Freeze
Step 2: Transition Plan (TP) — generated
         │ Validate → Freeze
Step 3: Readiness Confirmation (RC) — collected
         │ Validate → Freeze
         │
Output: RC readiness declaration informs REK entry decision
```

## Boundary Contracts

- **Upstream:** Receives frozen SAD and/or TDD from EEK (Layer 4). The trigger is: solution shape is known and affects business processes.
- **Downstream:** Produces a frozen RC whose readiness declaration informs the REK release entry decision. TP cutover schedule aligns with REK Release Plan timing.

## File Naming

| Type | Pattern |
|------|---------|
| Spec | `{type}-spec.md` |
| Template | `{type}-template.md` |
| Prompt | `{type}-prompt.md` |
| Validator | `{type}-validator.md` |

## When Working on This Kit

- Read the playbook (`docs/playbook.md`) for the full process definition
- Read the governance model (`docs/governance-model.md`) for structural rules
- Check `docs/how-to-use-with-ai.md` for session setup instructions

---
> Source: [wtlinnertz/aieos-business-process-kit](https://github.com/wtlinnertz/aieos-business-process-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
