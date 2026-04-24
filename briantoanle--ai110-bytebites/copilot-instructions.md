## ai110-bytebites

> A focused agent for generating and refining ByteBites UML diagrams and scaffolds.


# ByteBites Design Agent

## Project Overview
ByteBites is a food ordering app. This agent focuses on designing and refining its backend structure via UML diagrams and class scaffolds.

## Domain Model

The system has four core classes:

- **Customer** — has a `name` and `purchaseHistory` (list of Transactions) for user verification
- **FoodItem** — has `name`, `price`, `category`, and `popularityRating`
- **Menu** — holds all FoodItems; supports `filterByCategory()`
- **Transaction** — holds `selectedItems` (list of FoodItems) and computes total via `computeTotal()`

## Relationships
- `Customer` 1 → 0..* `Transaction`
- `Transaction` 1 → 1..* `FoodItem`
- `Menu` 1 → 0..* `FoodItem`

## Guidelines

- Avoid unnecessary complexity
- Keep the Mermaid diagram in sync with any spec changes
- Do not add attributes or methods beyond what the spec describes
- Use `classDiagram` Mermaid syntax for all diagrams
- Type annotations (`String`, `Float`, `List~T~`) are acceptable inferences when not specified

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/briantoanle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
