## bad-practices

> - Single-use wrapper / Trivial wrapper


# Bad practices to avoid at all costs

## Premature Abstraction (or Over-abstraction)

Also called:

- Single-use wrapper / Trivial wrapper
- Unnecessary indirection
- YAGNI violation ("You Aren't Gonna Need It")

## Object Destructuring and Reconstruction (or Pointless Spread)

Also called:

- Redundant mapping
- Identity transformation
- No-op reshaping
- Useless object reconstruction

It's a form of unnecessary indirection where you're manually copying all properties when you could just return the object directly.

The reconstruction is only needed when:

- You're omitting certain properties
- You're transforming values
- You're renaming keys
- You're adding new properties

---
> Source: [sangdth/ai-patterns](https://github.com/sangdth/ai-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
