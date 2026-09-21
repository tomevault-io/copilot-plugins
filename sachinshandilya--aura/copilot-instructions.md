## rules

> These rules help maintain clean, scalable, and consistent React code.

# React 18 Best Practices Rules

These rules help maintain clean, scalable, and consistent React code.

---

## File & Component Size
- Each React component file **must not exceed 300 lines**.
- Each component function **must not exceed 150 lines**.
- Avoid having more than **1 default export per file**.
- Split large components into smaller ones if they exceed these limits.

---

## Component Structure
- Always use **function components** with hooks, not class components.
- Component names must start with a **capital letter** and be in **PascalCase**.
- Files containing React components should be named in **PascalCase** (e.g., `UserCard.tsx`).

---

## Hooks & State Management
- Only call hooks at the **top level** of components or custom hooks.
- Always prefix custom hooks with `use` (e.g., `useAuth`, `useFetch`).
- Prefer **`useReducer`** for complex state logic.
- Do not use state if the value can be derived from props.
- Cleanup all side effects inside `useEffect`.

---

## Props & Types
- Always use **TypeScript** for props and state definitions.
- Use `interface` or `type` for props instead of `any`.
- Use **default values** or **PropTypes** (if JS-only) for optional props.
- Destructure props in function parameters.

---

## JSX & Rendering
- Keep JSX **pure and declarative**; avoid inline complex logic.
- No more than **3 levels of nested JSX**.
- Avoid inline functions in render unless memoization is used.
- Use **React.Fragment (`<>...</>`)** instead of unnecessary `<div>` wrappers.
- Keys in lists must be **stable and unique** (never use array index unless static).

---

## Performance
- Use `React.memo` for pure components that re-render frequently.
- Use `useCallback` and `useMemo` to avoid unnecessary re-renders.
- Code-split large routes/components with `React.lazy` + `Suspense`.
- Avoid passing unnecessary objects/functions as props.

---

## Imports & Project Structure
- Group imports: React, libraries, internal components, styles.
- Use absolute imports (via `tsconfig.json` paths) instead of deep relative imports.
- Do not import from `../..` when an alias can be configured.

---

## Styling
- Prefer **CSS Modules**, **Tailwind**, or **styled-components**.
- Keep component-specific styles colocated with the component.
- Avoid inline styles except for dynamic values.

---

## Error Handling
- Wrap async calls with `try/catch`.
- Use **Error Boundaries** for components that may throw.
- Never swallow errors silently.

---

## Testing
- All components must have **at least one test** (unit or integration).
- Prefer **React Testing Library** over Enzyme.
- Mock APIs and external services in tests.

---

✅ Follow these rules for clean, maintainable React 18 code.

---
> Source: [sachinshandilya/aura](https://github.com/sachinshandilya/aura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
