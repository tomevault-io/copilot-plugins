## das

> For any task assigned as UX/UI work, changes must be limited to presentation only.

# Project Guardrails

## UX/UI Only Rule

For any task assigned as UX/UI work, changes must be limited to presentation only.

Allowed changes:
- Visual styling
- Layout spacing and alignment
- Typography, color, shadows, borders, icons
- Responsive behavior
- Microcopy and labels
- Animations and transitions
- Non-functional component composition for presentation purposes

Strictly forbidden without explicit user approval:
- Business logic changes
- State management changes
- Data flow changes
- Routing changes
- API contract changes
- Database/schema changes
- Store/reducer/action changes
- Chart/query/calculation logic changes
- File structure refactors
- Dependency changes
- Performance/security refactors unrelated to presentation

## Working Rule

If a requested UI adjustment appears to require logic or structural edits, stop and ask first.

Default assumption:
- Preserve behavior exactly as-is
- Do not change outputs, side effects, or interaction logic
- Treat this repository as "style-only" unless the user clearly says otherwise

## Editing Preference

Prefer changing:
- CSS
- className usage
- markup ordering only when behavior stays identical
- design tokens and presentation props

Avoid changing:
- hooks behavior
- reducers/stores
- utility functions
- data transformations
- network calls

## UI Change Checklist

Before starting:
- Confirm this task is presentation-only
- Do not touch logic, state, routing, API, schema, or utilities
- Keep existing behavior identical unless explicitly approved

While editing:
- Prefer CSS, tokens, spacing, typography, color, and layout adjustments
- Keep DOM changes minimal and behavior-neutral
- Preserve existing component props and interaction flow
- Do not rename files, move files, or refactor architecture
- Do not add or remove dependencies

Before finishing:
- Verify the target screen still renders correctly
- Check desktop and mobile layouts
- Check overflow, spacing, and visual hierarchy
- Check hover, focus, active, disabled, and loading states if present
- Check text readability and contrast
- Confirm no console errors were introduced
- Confirm no data behavior changed

Definition of done for UI-only work:
- Same behavior
- Better clarity
- Better consistency
- Better responsiveness
- No structural side effects

---
> Source: [Minikot88/das](https://github.com/Minikot88/das) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
