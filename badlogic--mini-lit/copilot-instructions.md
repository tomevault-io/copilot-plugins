## mini-lit

> Biome is configured to require braces around arrow function bodies. Always use braces, even for single expressions:

# Claude Code Guidelines

## Code Style

### Arrow Functions
Biome is configured to require braces around arrow function bodies. Always use braces, even for single expressions:

```typescript
// ✅ Correct
array.forEach(item => {
  doSomething(item);
});

// ✅ Correct
const result = items.map(x => {
  return x * 2;
});

// ❌ Incorrect - will fail Biome linting
array.forEach(item => doSomething(item));
const result = items.map(x => x * 2);
```

This applies to all arrow functions including callbacks, effects, and event handlers.

---
> Source: [badlogic/mini-lit](https://github.com/badlogic/mini-lit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
