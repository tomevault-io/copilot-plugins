## design-system-jusbiz

> description: Cursor Rules for Custom Hooks

---
description: Cursor Rules for Custom Hooks
globs: *.ts,lib/hooks/**/*.ts
alwaysApply: false
---
# Cursor IDE Prompt Rules for Custom Hooks

## Hook Structure Rules
- Always use `use` prefix in hook names (e.g., `useResizeObserver`)
- Encapsulate hook logic in a single function
- Avoid side effects outside of `useEffect` or `useLayoutEffect`
- Use `useRef`, `useCallback`, or `useMemo` where appropriate to optimize performance
- If applicable, return only necessary data/functions, not internal refs or timeouts
- Prefer explicit return types for all public hooks
- Export hook as a named function

## Naming Conventions
- Use camelCase for hook names and internal variables
- Use descriptive names for arguments (e.g., `callback`, `delay`, `targetRef`)
- Prefix boolean flags with "is", "has", or "should"

## Documentation Rules
- Add JSDoc comments above the hook declaration
- Always document:
  - Purpose and behavior of the hook
  - Parameters and return values
  - Example usage in a functional component
- Explain optional configuration patterns (e.g., debounce options like `leading`)
- Include edge cases or gotchas when relevant (e.g., cleanup behavior)

## Types and Safety
- Avoid using `any`; use specific or generic types (`<T>` if needed)
- Clearly type all arguments and return values
- Type ref values correctly (e.g., `useRef<NodeJS.Timeout | null>`)
- Use `readonly` where immutability is intended

## File Structure
- Place each hook in its own file inside `lib/hooks/`
- Group reusable helper functions in adjacent utils files if logic is shared across hooks

## Advanced Patterns (when applicable)
- Support cleanup with `useEffect` return
- Allow optional configuration objects for hooks with more than 2 parameters
- Consider exposing `.cancel()` or `.flush()` methods for debounced/throttled hooks
- Use `AbortController` pattern for async hooks that involve fetch or subscriptions

## Hook Template Example

```ts
/**
 * useTimeout
 * 
 * A React hook that invokes a callback after a specified delay.
 *
 * @param {() => void} callback - Function to call after timeout.
 * @param {number} delay - Delay in milliseconds.
 *
 * @example
 * useTimeout(() => {
 *   console.log("Fired!");
 * }, 1000);
 */
export function useTimeout(callback: () => void, delay: number) {
  const savedCallback = useRef(callback);

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    const id = setTimeout(() => savedCallback.current(), delay);
    return () => clearTimeout(id);
  }, [delay]);
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juspay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
