## orbit-agent-safety-and-ui-consistency

> - Treat the latest user instruction as highest priority and execute exactly what was asked.

# Orbit Agent Safety and UI Consistency

## Instruction Priority
- Treat the latest user instruction as highest priority and execute exactly what was asked.
- If requirements conflict or are ambiguous, ask a focused clarification question before changing code.

## Edit Safety Gate
- Do not edit files with partial context. First inspect all directly related files, types, and call sites.
- Prefer no edit over a risky edit when confidence is low.
- Only apply changes once intent, impact, and integration points are clear.

## Reuse Before Creating
- Reuse existing Orbit React components and wrappers before adding new components.
- Follow established composition patterns in `src/vs/workbench/contrib/orbit/browser/react/src/sidebar-tsx/components/` and existing mapping/registry patterns.
- Extend existing helpers/constants when possible instead of duplicating logic.

## Color and Styling Rules
- Use existing theme tokens and CSS variables (for example `--void-*` and `--vscode-*`) from `src/vs/workbench/contrib/orbit/browser/react/src/styles.css`.
- Avoid hardcoded colors unless there is a strong, documented exception.
- Keep styling consistent with existing utility/class patterns already used in Orbit React UI.

## Responsiveness and Accessibility
- Preserve existing responsive behavior; do not regress small-screen layouts.
- When adding new UI, include responsive behavior consistent with current breakpoints and overflow handling patterns.
- Keep keyboard/focus behavior and reduced-motion support aligned with existing styles.

## Verification Before Finalizing
- Validate that changes do not break responsive layouts or existing component contracts.
- Run targeted checks/lints/tests when available for touched areas.
- If verification cannot be completed, clearly state what was not verified and why.

---
> Source: [ashish200729/orbiteditor](https://github.com/ashish200729/orbiteditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
