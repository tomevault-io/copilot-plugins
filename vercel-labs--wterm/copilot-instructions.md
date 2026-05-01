## avoid-useeffect

> Avoid useEffect in React/Next.js components


# Avoid useEffect

Do not use `useEffect` unless absolutely necessary. Modern React and Next.js provide better alternatives for nearly every case:

- **Data fetching** → Server Components, `use()`, or React Query
- **Derived state** → Compute during render or use `useMemo`
- **Responding to events** → Handle in event handlers directly
- **Subscribing to external stores** → `useSyncExternalStore`
- **Resetting state on prop change** → Use a `key` prop on the component
- **DOM refs / imperative setup** → `useRef` with callback refs

If `useEffect` truly cannot be avoided (e.g., initializing a non-React library that requires DOM access), document why with a comment.

---
> Source: [vercel-labs/wterm](https://github.com/vercel-labs/wterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
