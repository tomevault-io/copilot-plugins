## typescript-strict-typing

> Enforce strict TypeScript typing without any types


# TypeScript Strict Typing

**Never use `any` type.** Use `unknown` for dynamic data, proper interfaces for complex objects, and specific union types for known value sets.

### Examples:
```typescript
// ❌ Bad
function processData(data: any) { ... }
const context: Record<string, any>;

// ✅ Good
type LogContext = Record<string, unknown>;
interface Config { language: string; framework: string; }
type Status = 'success' | 'error' | 'pending';
```

---
> Source: [langwatch/better-agents](https://github.com/langwatch/better-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
