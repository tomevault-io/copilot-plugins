## frontend-app-template

> - Use React versions 19 only (no higher versions).


## React Rules

- Use React versions 19 only (no higher versions).
- Use functional components and hooks, avoiding class-based components except for error handling cases like `ErrorBoundary`.
- Follow the Rules of Hooks (call hooks at the top level, only in React functions).
- Minimize reliance on `useState` and `useEffect`; prefer custom hooks for reusable logic.
- Use `React.memo` for memoizing components (with display names), `useCallback` for functions passed as props, and `useMemo` for expensive computations; avoid premature optimization.
- Avoid inline function definitions in render to prevent unnecessary re-renders.
- Prefer composition over inheritance and use `children` prop or render props for flexibility.
- Implement `React.lazy()` and `Suspense` for code splitting with fallback UI.
- Use refs sparingly, only for DOM access, and prefer controlled components over uncontrolled ones.
- Implement cleanup in `useEffect` to prevent memory leaks.
- Use short-circuit evaluation (`&&`) and ternary operators for conditional rendering; ensure the left side of `&&` is always a boolean value (e.g., `isActive && <Component />`); prefer explicit conditions for complex logic.

## TypeScript Rules

- Use the latest stable version of TypeScript.
- Use `interface` for defining props and object-like structures. Use `type` for unions, intersections, and other complex types.
- Ensure strict typing for all components, functions, and variables; **never use `any` and `as` type**; use proper type guards and generics.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ardor-Cerebrum) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
