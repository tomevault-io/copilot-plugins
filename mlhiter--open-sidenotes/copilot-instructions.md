## open-sidenotes

> This file is the working agreement for Codex in this repository.

# AGENTS.md

This file is the working agreement for Codex in this repository.
It defines product direction, implementation guardrails, and delivery standards.

## Mission

- Keep Open Sidenotes a notes-first app for fast, low-friction capture.
- Favor elegant simplicity over feature breadth.

## Product Direction

- Keep Notes as the primary mode and avoid branching into secondary workspaces.
- Remove or de-emphasize task-module style UX from the main product flow.
- Preserve quick capture behavior: open, type, save should feel immediate.
- Improve polish and reliability before adding peripheral features.

## UX Guardrails

- Settings must remain easy to reach from the notes workspace.
- Avoid deep navigation for frequent actions.
- Reduce visual noise: clear hierarchy, deliberate spacing, simple controls.
- Prefer fewer stronger UI elements over dense control surfaces.

## Current Priorities (Ordered)

1. Notes capture/editing speed and reliability.
2. Navigation simplification in the notes workflow.
3. Visual refinement with minimal complexity.
4. Structural refactors that improve maintainability without UX regression.

## Scope Guardrails

- Do not re-introduce task-manager-centric IA as a primary experience.
- Do not add complex multi-step flows for common actions.
- Do not ship feature-heavy UI that dilutes the notes-first focus.
- For any new feature, require a clear fit with the notes-first workflow.

## Engineering Preferences

- Refactors are encouraged when they improve clarity, reliability, or UX.
- Prefer iterative but meaningful improvements over cosmetic-only tweaks.
- Keep the app buildable after each meaningful batch of edits.
- Preserve existing behavior unless a deliberate UX change is intended.
- Keep changes coherent: avoid unrelated edits in the same milestone.

## Implementation Standards

- Make the smallest architecture change that cleanly solves the problem.
- Maintain consistent naming and clear separation of responsibilities.
- Add or adjust tests when touching non-trivial logic.
- For UI work, verify both desktop ergonomics and small-window behavior.
- Document new constraints or patterns in this file when they become recurring.

## Quality Gates

- Build must pass for the main scheme before final handoff.
- For behavior changes, include a short manual verification checklist in updates.
- Flag known tradeoffs or follow-ups explicitly instead of hiding them.
- Avoid speculative rewrites without measurable product or maintenance benefit.

## Collaboration Rules

- Codex may commit proactively without asking each time.
- Codex decides autonomously whether to commit and push based on task scope and risk.
- Commit at logical, testable milestones.
- Use clear commit messages describing user-visible outcomes.
- If unexpected repo changes appear mid-task, stop and ask before proceeding.

## Commit Message Style

- Use concise, outcome-focused subjects.
- Prefer format: `<area>: <user-visible change>`.
- Include brief body notes when behavior, migration, or tradeoffs matter.

## Definition of Done

- Change aligns with product direction and UX guardrails above.
- Relevant build/tests pass, or any unrun checks are explicitly called out.
- No obvious regression in the Notes-first flow.
- Documentation is updated when behavior or conventions changed.

## Living Document

- Update this file when priorities shift or new recurring constraints emerge.
- Keep guidance practical and enforceable; remove stale rules quickly.

---
> Source: [mlhiter/open-sidenotes](https://github.com/mlhiter/open-sidenotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
