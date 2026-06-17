## ui-scout

> Use before building any UI component from scratch to search existing component sources, libraries, and AI generators first.


# UI Scout

**Core rule:** search before you build. A component that already exists shouldn't be rebuilt.

---

## The Search Order

Before writing any new UI component:

1. **21st.dev MCP** — use `mcp__21st-dev-magic__21st_magic_component_builder` or `mcp__21st-dev-magic__21st_magic_component_inspiration` if available
2. **shadcn/ui** — check [ui.shadcn.com/components](https://ui.shadcn.com/components)
3. **Aceternity UI** — for animated, modern components
4. **Radix UI** — for headless, accessible primitives
5. **Magic UI** — for animated components and effects
6. **Build from scratch** — only if none of the above is ≥80% of what's needed

## The 80% Rule

If an existing component covers 80% or more of the requirement → adapt it, don't rebuild.

Adapting means adjusting styles, adding a prop, or composing two existing components.
Adapting does NOT mean rewriting the core logic from scratch.

## When Using MCP Tools

When 21st.dev MCP is available:
- `21st_magic_component_builder` — build new components
- `21st_magic_component_inspiration` — discover similar components
- `21st_magic_component_refiner` — iterate on existing components

## Attribution

When adapting a component, add a source comment:
```
// Adapted from shadcn/ui Button
// https://ui.shadcn.com/components/button
```

---

## Anti-Patterns

| Pattern | Why It's Harmful |
|---------|----------------|
| Building a modal from scratch when Radix Dialog exists | Reinvents accessibility work |
| Rebuilding shadcn components with minor style changes | Unnecessary maintenance burden |
| Skipping the search because "it's faster to build" | It's never faster long-term |

---

## What Stays the Same

Component quality, accessibility, and design fidelity are never compromised.

---

## Manual Activation

Invoke with `/ui-scout` before any UI component implementation session.

---
> Source: [Feli2arias/ui-scout](https://github.com/Feli2arias/ui-scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
