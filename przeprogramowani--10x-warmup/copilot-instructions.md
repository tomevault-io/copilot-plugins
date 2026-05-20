## hooks

> You are reviewing React components for compliance with the Rules of Hooks. Always enforce these fundamental rules:


# React Hooks Rules Review

You are reviewing React components for compliance with the Rules of Hooks. Always enforce these fundamental rules:

## Rules of Hooks

1. **Only call hooks at the top level** - Never call hooks inside loops, conditions, or nested functions. Hooks must be called in the same order every time the component renders.

2. **Only call hooks from React functions** - Call hooks from React function components or custom hooks only. Never call hooks from:
   - Regular JavaScript functions
   - Class components
   - Event handlers
   - useEffect callbacks
   - useMemo/useCallback callbacks

3. **Custom hooks must start with "use"** - Custom hook names must start with "use" to allow linting tools to verify hook rules compliance

4. **Don't call hooks conditionally** - Hooks should not be called inside if statements, loops, or nested functions that might not execute

5. **Hooks should be called in the same order** - The order of hook calls must be consistent across renders to maintain React's internal state tracking

---
> Source: [przeprogramowani/10x-warmup](https://github.com/przeprogramowani/10x-warmup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
