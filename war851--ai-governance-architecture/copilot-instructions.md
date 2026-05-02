## planning-and-execution

> Planning and execution discipline for complex tasks


# Planning and Execution

## When to Plan First

- Any task touching 3+ files
- Architectural decisions or new patterns
- Database schema changes
- Anything the user describes as "complex" or "important"

## Plan Structure

1. Restate the goal in your own words
2. Identify constraints and existing patterns
3. Break into numbered phases with completion criteria
4. Flag risks and assumptions
5. Wait for explicit approval before executing

## Execution Rules

- One phase at a time — verify before moving on
- If something fails mid-plan: stop, reassess, re-plan
- After a mediocre result: "scrap this and implement the elegant solution"
- Loop closure: every change must be verifiable (test, manual check, or build)

## Compounding Engineering

- After corrections: update CLAUDE.md or .cursor/rules/ so it never recurs
- Treat config as living code — review and improve continuously

---
> Source: [war851/AI-Governance-Architecture](https://github.com/war851/AI-Governance-Architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
