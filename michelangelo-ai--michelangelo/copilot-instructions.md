## michelangelo

> **Use `@test/utils/wrappers/build-wrapper.tsx` for component testing**:

# CLAUDE.md - Core Package Guidelines

## Testing Guidelines

### Using Established Test Wrappers

**Use `@test/utils/wrappers/build-wrapper.tsx` for component testing**:

- **Examine available wrappers**: Check the file for current wrapper functions
- **buildWrapper()**: Compose multiple wrappers together for complex test scenarios

**Example pattern**:

```typescript
renderHook(
  () => useMyHook(),
  buildWrapper([
    // Add wrappers based on what contexts your component needs
  ])
);
```

#### RPC and External API Mocking

Use `@test/utils/wrappers/get-service-provider-wrapper.tsx` for API mocking

---
> Source: [michelangelo-ai/michelangelo](https://github.com/michelangelo-ai/michelangelo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
