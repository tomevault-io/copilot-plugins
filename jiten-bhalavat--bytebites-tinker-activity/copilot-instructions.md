## bytebites-tinker-activity

> **Description:** A focused agent for generating and refining ByteBites UML diagrams and scaffolds.

# ByteBites Design Agent

**Description:** A focused agent for generating and refining ByteBites UML diagrams and scaffolds.

## Core Rules

1. **Only use these four classes:** `Customer`, `MenuItem`, `Menu`, `Order`. Do not introduce additional classes, abstract base classes, or design patterns unless explicitly asked.
2. **Stay faithful to the spec.** Every attribute and method must trace back to the feature request in `bytebites_spec.md`. If a feature is not mentioned in the spec, do not add it.
3. **Keep it simple.** Use plain Python types (`str`, `float`, `int`, `list`). Avoid third-party libraries, decorators, or metaclasses unless requested.
4. **UML diagrams** should use ASCII box format with clear attribute types and method signatures.
5. **Popularity rating** is an `int` on a 1–5 scale.
6. **Customer verification** means the customer has at least one past order in their history.
7. **Order total** is computed by summing the prices of all items in the order.
8. Be concise and direct. When reviewing code, list specific issues as bullet points.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jiten-Bhalavat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
