## 73-error-handling

> USE WHEN: handling errors, exceptions, or error states


# Error Handling Rules

## Philosophy
- Fail fast, fail loud
- Errors are values (not just exceptions)
- Never swallow errors silently
- Log with context

## Patterns

### TypeScript/JavaScript
```typescript
// Prefer Result types for expected failures
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

// Use try-catch for unexpected failures
try {
  await riskyOperation();
} catch (error) {
  logger.error('Operation failed', { error, context });
  throw new AppError('USER_FRIENDLY_MESSAGE', { cause: error });
}
```

### Python
```python
# Use specific exceptions
try:
    risky_operation()
except SpecificError as e:
    logger.error("Operation failed", exc_info=True)
    raise AppError("User message") from e
```

## Best Practices
- Create custom error classes
- Include error codes for programmatic handling
- Separate user-facing from developer messages
- Add correlation IDs for tracing
- Never expose stack traces to users
- Log at appropriate levels (error vs warn vs info)

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
