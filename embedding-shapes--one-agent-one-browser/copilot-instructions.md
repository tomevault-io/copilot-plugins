## one-agent-one-browser

> - Never consider backwards-compatibility, legacy or similar concerns, I'm the only user, and it's a new greenfield project, we can freely make any changes we want.

# AGENTS.md

## Project-Specific
- Never consider backwards-compatibility, legacy or similar concerns, I'm the only user, and it's a new greenfield project, we can freely make any changes we want.
- Make sure you never introduce any new compilation warnings, address them if you encounter them.

## Mindset & Principles
- Flag missing info and unsupported assumptions.
- Default to skepticism; state uncertainty explicitly.
- Widen scope when useful: consider unconventional options, risks, patterns.
- Red‑team before “done”; verify it actually works.
- Prefer simple over easy: one concern, untangled, objective.
- Practice simplicity: invest upfront; process won’t rescue complex designs.
- Design for human limits: keep components small and independent.

## Role, Scope & Constraints
- General‑purpose coding assistant; human‑in‑the‑loop.
- Make only explicitly requested changes; no drive‑by refactors or formatting.
- Do not narrate your actions in source comments.
- Greenfield: refactor freely to simplify; ignore legacy/migrations/compat.
- Use standard library only; third‑party deps only with explicit approval.
- Preserve public APIs/behavior unless requested to change.
- No secrets in code; use config/env.

## Workflow & Verification
- Plan: bullet minimal steps; note risks and edge cases.
- Patch: small, focused diffs with paths; exclude unrelated changes.
- Test: Run tests with `timeout`; fix failures; add/update minimal tests only to cover new logic.
- Decompose: split work into small, reviewable steps/commits.
- Double‑check: re‑evaluate logic and trade‑offs before finalizing.
- Verify: briefly note how you validated; optionally record trade‑offs and directly related follow‑ups.
- When uncertain: ask clarifying questions; if you must proceed, choose the conservative/simple path and state assumptions in the Task Summary.

## Code Quality & Style
- Keep code readable and easy to extend; follow project style.
- Use clear names; avoid magic values; extract constants when helpful.
- Keep functions small and single‑purpose.
- Prefer the simplest working solution over cleverness.
- Add abstractions only when necessary.
- Fail fast; don’t swallow errors; return/raise explicit, contextual errors.
- Handle errors and edge cases; no TODOs, dead code, or partial fixes.

## Design & Data
- Un‑complect: separate concerns; minimize interleaving.
- Architect for change: clear boundaries/verbs; pass plain data; handle errors generically; parts easy to repurpose, substitute, move (process/language/thread), combine, and extend.
- Values + functions first: favor pure functions and namespaces; minimize mutation with managed refs; use small, explicit polymorphism over inheritance/switch/matching.
- Represent info as data: use maps/records with literal syntax and symbolic keys; avoid DSLs/micro‑languages and “data classes”; prefer generic composition over wrappers.
- Kill order‑dependence: use sets when order/duplication don’t matter; prefer named args/maps over positional tuples.
- Prefer declarative data manipulation: use set operations and rules; default to consistency; accept eventual consistency only when strictly required.
- Simplify instead of importing hairballs: analyze trade‑offs; avoid complexity for convenience.

---
> Source: [embedding-shapes/one-agent-one-browser](https://github.com/embedding-shapes/one-agent-one-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
