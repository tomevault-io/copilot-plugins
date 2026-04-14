## news-bot-fe

> Component Architecture

React Best Practices

Component Architecture
- Use functional components with TypeScript interfaces
- Define components using arrow function
- Extract reusable logic into custom hooks
- Implement proper component composition
- Implement proper cleanup in useEffect hooks
- Do not use React.FC or similar wrapper types for function components
- Do not wrap components with React.memo by default

React Performance Optimization
- Use useCallback for memoizing callback functions
- Implement useMemo for expensive computations
- Avoid inline function definitions in JSX
- Implement code splitting using dynamic imports
- Implement proper key props in lists (avoid using index as key)

State and Data Flow
- Keep state minimal and colocated; lift only when multiple consumers
  need it.
- Prefer derived data via useMemo instead of duplicating source state.

Effects and Async
- Ensure useEffect dependency arrays are complete and stable; prefer
  useCallback/useMemo to avoid re-renders.
- A useEffect with an empty dependency array is allowed only once per 
  component and must be placed after all other useEffect hooks
- Cancel async work on unmount (e.g., AbortController) to prevent
  leaks and state updates on unmounted components.
- Debounce/throttle event-driven effects that can fire frequently
  (scroll, resize, keypress).

Accessibility and UX
- Use semantic HTML and appropriate ARIA roles/labels; ensure keyboard
  navigation for all interactive elements.
- Manage focus on dialogs, toasts, and route changes; return focus to
  the triggering control when closing overlays.
- Provide visible focus states and avoid removing outline without an
  accessible alternative.
- Prefer progressive enhancement: render meaningful loading and empty
  states (skeletons/spinners/placeholder copy).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Romkoos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Romkoos)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
