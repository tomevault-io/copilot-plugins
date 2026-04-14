## happyrow-core

> Stack-agnostic frontend conventions - components, separation of concerns, accessibility


# Frontend Conventions

## Component structure

- One component per file, named in **PascalCase**: `UserProfile`, `OrderList`
- Separate **presentational** components (render UI, receive data via props) from **container** components (manage state, call use cases)
- Keep components small and focused on a single responsibility

## Business logic

- NEVER put domain logic inside UI components
- Components call application-layer use cases or services
- If logic is reused across components, extract it into the `application/` or `domain/` layer

## Styles

- Colocate styles with their component (same folder or same file depending on stack)
- Use consistent naming: BEM, CSS modules, or scoped styles — pick one per project and stick with it

## Accessibility

- Semantic HTML elements over generic `div`/`span` (`nav`, `main`, `button`, `article`)
- Interactive elements must be keyboard-navigable
- Images require meaningful `alt` text (or `alt=""` for decorative)
- Form inputs must have associated `label` elements
- Sufficient color contrast (WCAG AA minimum)

## State management

- Keep state as close to where it's used as possible
- Avoid global state for data that only one component needs
- Side effects (API calls, storage) belong in the infrastructure layer, not in components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimni6)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jimni6)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
