## radzionkit

> Trust TS-defined values – avoid optional chaining and fallbacks when types guarantee presence


## Trust TypeScript – No Fallbacks for Defined Values

When a value's type is non-optional, use it directly. Do not add optional chaining or fallback values.

Do:

```ts
const { requestOrigin } = usePopupContext()
useIt(requestOrigin)
```

Don't:

```ts
const ctx = usePopupContext()
useIt(ctx?.requestOrigin || '')
```

Notes:

- If a value may be missing, validate earlier and fail fast (e.g., assertions in @rules).
- Do not silence type guarantees with `?.`, `||`, or `??` when the type is non-optional.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/radzionc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
