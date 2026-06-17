## component-contract

> Use before implementing any UI component to define its API, states, and accessibility requirements upfront.


# Component Contract

**Core rule:** define the component's contract before writing a single line of implementation.

---

## The Contract (Required)

Write this as a comment block at the top of the component file:

```
// Contract: ComponentName
// Props: [list all props with types and whether required]
// Variants: [all visual variants]
// States: default | hover | active | disabled | loading | error | empty
// Accessibility: ARIA role, keyboard interaction, contrast requirement
// Responsive: [behavior at sm / md / lg breakpoints]
```

## Rules

- No implementation until the contract is written
- All states must be accounted for — a missing state is a bug
- Accessibility is part of the contract, not an afterthought
- Responsive behavior defined per breakpoint — not added later
- Props API is stable once implementation starts

## What Counts as a State

**Always required:** default, hover, active/focus, disabled
**When applicable:** loading, error, empty/zero-data, success

---

## Anti-Patterns

| Pattern | Why It's Harmful |
|---------|----------------|
| Adding props as implementation progresses | Unstable API, breaking changes |
| "I'll add disabled state later" | It never gets done |
| Accessibility as a post-implementation checkbox | Retrofitting a11y is always harder |
| Responsive as an afterthought | Layout breaks on mobile |

---

## What Stays the Same

Visual quality and implementation correctness are never compromised.

---

## Manual Activation

Invoke with `/component-contract` before any UI component implementation.

---
> Source: [Feli2arias/component-contract](https://github.com/Feli2arias/component-contract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
