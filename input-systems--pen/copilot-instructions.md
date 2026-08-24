## pen-headless-react-primitives

> React rendering primitives stay headless and composable


# Headless React Primitive Conventions

- Keep React primitives unstyled and behavior-focused; do not bake in design system choices.
- Maintain composable primitive API shape (Root, Trigger, Content, Group, Item, etc.) aligned with spec.
- Preserve `asChild` and ref-forwarding patterns where composition requires DOM control transfer.
- Expose stable `data-*` state attributes for styling/automation instead of internal class assumptions.
- Keep framework package boundaries clean: renderer code should consume core interfaces, not own core logic.

---
> Source: [input-systems/pen](https://github.com/input-systems/pen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
