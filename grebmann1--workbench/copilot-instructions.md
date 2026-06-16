## agent-simplicity-and-dedup

> Reuse local patterns, reduce duplication, and keep changes narrowly scoped


# Agent Simplicity And Dedup

- Read nearby files first and match local naming, structure, and abstraction patterns before inventing new ones.
- Prefer reusing existing utilities, components, constants, and modules over copying logic into a second place.
- If code is duplicated, first look for an existing helper or a small extraction that improves clarity without widening scope.
- Keep diffs tightly scoped to the request and avoid opportunistic refactors that add risk without clear value.
- Choose the simplest implementation that satisfies the request with the fewest new branches, layers, and runtime costs.

---
> Source: [grebmann1/workbench](https://github.com/grebmann1/workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
