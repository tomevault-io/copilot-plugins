## openchat

> - Stores handle complex persisted state (e.g., theme editor).


# State Management Review Guidelines

## Zustand Stores (`src/lib/store/`)
- Stores handle complex persisted state (e.g., theme editor).
- Keep store slices focused — don't create god stores.
- Derived state should be computed selectors, not stored separately.
- Avoid storing server state in Zustand — use TanStack Query + Convex for that.

## React Context Providers (`src/providers/`)
- Providers handle app-wide state (ChatSession, Sidebar, User).
- Keep provider values stable — memoize with `useMemo` to prevent unnecessary re-renders.
- Don't put frequently-changing values in context — it re-renders all consumers.
- Split contexts by update frequency if needed.

## Data Fetching
- Use `convexQuery()` wrapper for Convex server state.
- TanStack Query for non-Convex async state.
- Never duplicate server state in local state — subscribe to the source of truth.

---
> Source: [ajanraj/OpenChat](https://github.com/ajanraj/OpenChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
