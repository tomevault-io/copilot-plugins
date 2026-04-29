## tui

> TypeScript-first standards for reusable frontend MUI components


# Frontend TypeScript Component Standards

Use TypeScript as the canonical implementation for reusable components before creating JavaScript parity versions.

## Type Safety Requirements

- Define explicit `Props` types or interfaces for every exported component.
- Avoid `any`; use unions, generics, and utility types where appropriate.
- Type event handlers explicitly for React and MUI component usage.
- Export prop types when they are reused by wrapper or composed components.

## Reusable API Design

- Prefer small, composable prop surfaces over large ad hoc option bags.
- Use clear defaults and optional props instead of implicit behavior.
- Keep variant naming and prop semantics consistent across component families.

## MUI + Tailwind Parity Requirements

- Implement visual parity with Tailwind references using MUI `sx`, props, and theme tokens.
- Keep responsive and interactive behavior equivalent.
- Ensure the TS implementation is the source for later JS conversion with matching runtime behavior.

## Quality Guardrails

- Keep component logic separated from presentation helpers when complexity grows.
- Preserve accessibility semantics (`aria-*`, role, focus behavior) with typed props.
- Document any unavoidable visual or behavioral deltas from the Tailwind reference.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VideoGameRoulette) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
