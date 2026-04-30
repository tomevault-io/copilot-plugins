## core-policy

> CorePolicy uses a three-module architecture:

# CorePolicy Agent Instructions

## Architecture
CorePolicy uses a three-module architecture:
- `:app`: Main entry point and lifecycle management.
- `:ui-skeleton`: Pure UI components and screens.
- `:core-logic`: Data sets, models, and background logic.

## Stability Guidelines
1. **Structured Concurrency**: Always use `viewModelScope` or `lifecycleScope`. Avoid `GlobalScope`.
2. **Error Handling**: Use `CoroutineExceptionHandler` for all top-level launches.
3. **DataStore Safety**: Always use `.catch { ... }` on DataStore Flows to handle `IOException`.
4. **Memory Awareness**: `AppIconCache` implements `ComponentCallbacks2`. Ensure any new large caches do the same.

## Reimplementation Focus
- Keep logic in `:core-logic`.
- Keep UI purely reactive to state from `:core-logic`.
- Information requiring privileges should fallback to "Unknown" if privileges are absent.
- Use `CoreLogger` for all background tasks.

---
> Source: [DikyVinus/core_policy](https://github.com/DikyVinus/core_policy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
