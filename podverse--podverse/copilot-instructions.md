## semicolons-required

> Require semicolons in JS/TS files only; .md and .mdc are exempt


# Semicolons Required

- Use semicolons at the end of statements in **JavaScript and TypeScript files only** (`.js`, `.jsx`, `.ts`, `.tsx`).
- **Exempt:** Markdown (`.md`) and Cursor rule (`.mdc`) files are not subject to this rule.
- When updating existing code, keep semicolons consistent with this rule.

```typescript
// ✅ Good
const value = 123;
doWork();

// ❌ Bad
const value = 123
doWork()
```

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
