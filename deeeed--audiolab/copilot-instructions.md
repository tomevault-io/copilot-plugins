## audiolab

> - Use `interface` over `types` when possible for better extensibility

# TypeScript Best Practices

## Type System

- Use `interface` over `types` when possible for better extensibility
- **Never use enums** - always use const objects instead:

  ```typescript
  // ❌ Avoid enums
  enum DeviceType {
    BUILTIN = 'builtin_mic',
    BLUETOOTH = 'bluetooth'
  }
  
  // ✅ Use const objects 
  const DeviceType = {
    BUILTIN: 'builtin_mic',
    BLUETOOTH: 'bluetooth'
  } as const;
  
  type DeviceTypeValue = typeof DeviceType[keyof typeof DeviceType];
  ```

- Prefer object parameters for functions with multiple parameters
- Use strict type checking
- Never use `any`, prefer types `object` or use `unknown`
- Do not create useless index.ts files that only re-export interfaces; import directly from source

## Architecture Patterns

- Apply platform abstraction for cross-platform code
- Create consistent APIs across different implementations
- Use clear error typing for better error handling

---
> Source: [deeeed/audiolab](https://github.com/deeeed/audiolab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
