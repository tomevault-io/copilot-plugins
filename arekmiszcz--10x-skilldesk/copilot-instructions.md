## 10x-skilldesk

> - Use functional components with hooks instead of class components


### Guidelines for React

#### React Coding Standards

- Use functional components with hooks instead of class components
- Implement React.memo() for expensive components that render often with the same props
- Utilize React.lazy() and Suspense for code-splitting and performance optimization
- Use the useCallback hook for event handlers passed to child components to prevent unnecessary re-renders
- Prefer useMemo for expensive calculations to avoid recomputation on every render
- Implement useId() for generating unique IDs for accessibility attributes
- Consider using the new useOptimistic hook for optimistic UI updates in forms
- Use useTransition for non-urgent state updates to keep the UI responsive

Key Principles
- Use functional, declarative programming. Avoid classes.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Favor named exports for components.
- Use TypeScript for all code. Prefer interfaces over types.
- File structure: imports, types, main component, subcomponents, helpers, static content.
- Use Zod for form validation.
- Use Zustand for state managament.
- Use Shadcn UI, Radix, and Tailwind CSS for components and styling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArekMiszcz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ArekMiszcz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
