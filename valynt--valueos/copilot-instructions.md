## valueos

> **Paths:** `src/tools/*` & `src/services/tools/*`


# Tool Libraries

**Paths:** `src/tools/*` & `src/services/tools/*`

- Tools must implement `Tool<TInput, TOutput>` interface
- Register in `ToolRegistry.ts` (dynamic creation FORBIDDEN)
- Check `LocalRules` (LR-001) before execution
- External API tools MUST use `RateLimiter` middleware

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Valynt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
