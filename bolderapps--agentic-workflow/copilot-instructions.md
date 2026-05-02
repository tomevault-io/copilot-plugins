## 04-design-fidelity

> Non-negotiable design fidelity gates for Figma-driven UI tasks.


# Design fidelity rules

Apply these rules to any task with `Design source: Figma`, `Codegen artifact`, or `Visual baseline refs`.

## 1) Token integrity

- Do not hardcode visual values when a design token exists.
- Required token categories:
  - color
  - typography
  - spacing
  - radius
  - shadow/elevation
- If a hardcoded value is unavoidable, include an inline reason and add a follow-up decision entry.

## 2) State parity

For each designed component/state matrix, implement all required states:
- default
- hover
- focus
- disabled
- loading
- error
- empty (if applicable)

Missing a designed state means task cannot pass QA.

## 3) Responsive parity

- Respect breakpoint behavior defined in design artifacts/cache.
- Do not collapse, hide, or reorder critical content outside defined responsive rules.
- Core user paths must remain usable at target viewport sizes documented by design.

## 4) Structure and semantics

- Keep scaffold hierarchy from codegen artifacts unless acceptance criteria require a change.
- Treat Figma device previews as framing only. Do not duplicate system status bars, notches, home indicators, or phone shells inside application UI unless the task explicitly covers app-shell chrome.
- Preserve semantic HTML and accessible names.
- Keyboard focus order and focus visibility are required for interactive controls.

## 5) Visual baseline checks

- If visual baseline refs exist, QA must run design-fidelity checks against them.
- Any major mismatch in layout/tokens/states must result in `needs-fix`.
- No UI task can be marked `done` without passing QA design-fidelity checks.

## 6) Builder/QA contract

- Builder must complete `Design checklist` in task block before handoff.
- QA must verify checklist objectively and reject vague self-assertions.
- A task with unresolved design checklist items is not eligible for `done`.

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
