## buddy

> Every `Effect.Service` class **automatically exports** a `.Default` layer that can be used for dependency injection in tests and applications.

# Effect.Service Default Layer Pattern

## 🔑 **Critical Rule: Effect.Service Auto-Exports .Default**

### Automatic Layer Export
Every `Effect.Service` class **automatically exports** a `.Default` layer that can be used for dependency injection in tests and applications.

```typescript
// Service Definition
export class MyService extends Effect.Service<MyServiceApi>()(
  "MyService",
  {
    effect: Effect.succeed({ /* implementation */ }),
    dependencies: []
  }
) {}

// ✅ CORRECT: .Default is automatically available
MyService.Default  // This exists automatically
```

### Test Usage Pattern
```typescript
// ✅ CORRECT: Use .Default in tests
it("should work correctly", () =>
  Effect.gen(function* () {
    const service = yield* MyService;
    const result = yield* service.doSomething();
    expect(result).toBe("expected");
  }).pipe(
    Effect.provide(MyService.Default), // Use .Default layer
    Effect.runPromise
  ));
```

### Scoped Services
For services using `scoped` instead of `effect`, the `.Default` layer handles the scope automatically:

```typescript
// Service with scoped
export class ScopedService extends Effect.Service<ScopedServiceApi>()(
  "ScopedService",
  {
    scoped: Effect.gen(function* () {
      // scoped implementation
    }),
    dependencies: []
  }
) {}

// ✅ CORRECT: .Default handles scoping
it("should work with scoped service", () =>
  Effect.gen(function* () {
    const service = yield* ScopedService;
    const result = yield* service.doSomething();
    expect(result).toBe("expected");
  }).pipe(
    Effect.provide(ScopedService.Default), // Handles scope automatically
    Effect.runPromise
  ));
```

## ❌ **Common Mistakes**

### DO NOT manually create layers
```typescript
// ❌ WRONG: Don't create manual layers
const MyServiceLive = Layer.succeed(MyService, /* implementation */)

// ❌ WRONG: Don't export separate layers
export const MyServiceLive = Layer.scoped(MyService)
```

### DO NOT use service class directly in provide
```typescript
// ❌ WRONG: Don't use service class directly
Effect.provide(MyService) // This is incorrect

// ✅ CORRECT: Use .Default layer
Effect.provide(MyService.Default)
```

## 🎯 **Best Practices**

1. **Always use `.Default`** in tests and dependency injection
2. **Never create manual layers** for Effect.Service classes
3. **Trust the automatic layer generation** - it handles scoping, dependencies, etc.
4. **Use consistent pattern** across all service tests

## 📝 **Test Template**
```typescript
import { Effect } from "effect";
import { describe, expect, it } from "vitest";
import { MyService } from "../MyService";

describe("MyService", () => {
  it("should perform operation", () =>
    Effect.gen(function* () {
      const service = yield* MyService;
      const result = yield* service.operation();
      expect(result).toBe("expected");
    }).pipe(
      Effect.provide(MyService.Default),
      Effect.runPromise
    ));
});
```

This pattern works for **all** Effect.Service classes, regardless of whether they use `effect`, `scoped`, or have dependencies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PaulJPhilp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
