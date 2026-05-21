## exception-handling

> Anti-pattern: Catching an exception only to log and rethrow it

# Exception Handling Guidelines

## Don't Catch Just to Log

Anti-pattern: Catching an exception only to log and rethrow it
```typescript
async function doSomething() {
  try {
    await someOperation();
  } catch (error) {
    console.error('Error:', error);  // Unnecessary logging
    throw error;  // Just rethrow
  }
}
```

Better: Let the exception propagate if you're not handling it
```typescript
async function doSomething() {
  await someOperation();
}
```

## When to Use Try-Catch

Only catch exceptions when you are:
1. Transforming the error into a more appropriate type
2. Adding critical context that would be lost otherwise
3. Performing cleanup or recovery operations
4. Actually handling the error (e.g., fallback behavior)

---
> Source: [rejot-dev/rejot](https://github.com/rejot-dev/rejot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
