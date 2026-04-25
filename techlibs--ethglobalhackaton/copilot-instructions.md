## ethglobalhackaton

> Guidelines for performance optimization, formatting, imports, and general React TypeScript best practices.


# React Performance & Best Practices

## Performance Optimization

- **Always use `useMemo` and `useCallback` as required to prevent unnecessary re-renders.**
  Memoize expensive calculations and stable function references for props and dependencies.

  ```tsx
  function ProductList({ products, onSelect }: ProductListProps) {
    const sortedProducts = React.useMemo(
      () => [...products].sort((a, b) => a.name.localeCompare(b.name)),
      [products]
    );

    const handleSelect = React.useCallback(
      (id: string) => onSelect(id),
      [onSelect]
    );

    return (
      <ul>
        {sortedProducts.map((product) => (
          <li key={product.id} onClick={() => handleSelect(product.id)}>
            {product.name}
          </li>
        ))}
      </ul>
    );
  }
  ```

- **Use React.memo for components that receive stable props.**
- **Optimize list rendering with proper keys.**
- **Avoid creating objects/functions in render.**

## Functional Programming Principles

- **Functional-first:** Use function components and hooks, not classes.
- **Immutability:** Treat state and props as immutable.
- **Use pure functions, stateless logic, and composition.**
- **Map/filter/reduce for list transformations.**
- **No side effects in render or pure functions.**

## Formatting & Naming Conventions

- **PascalCase for components, camelCase for variables/functions, UPPER_CASE for constants.**
- **2-space indentation.**
- **No semicolons (unless required). Single quotes for strings.**
- **Suffix custom hooks with `use*` and component files with `.tsx`.**

## Imports & Exports

- **No barrel files.**
  Always import components, hooks, or utils directly from their file.
- **Use relative paths or path aliases (e.g., `@/components/Button`).**
- **Use ES6 modules. Prefer named exports.**
  ```tsx
  import { UserCard } from '@/components/UserCard';
  import { useUser } from '@/hooks/useUser';
  ```

## Code Organization

- **Component files must export only one component.**
- **Split large components into smaller, focused ones.**
- **Group related functionality in custom hooks.**
- **Keep components focused on presentation.**

## Documentation

- **No inline comments or excessive JSDoc.**
  Code must be self-explanatory.
- **Use markdown for high-level docs only.**
- **Write descriptive component and prop names.**

## General Guidelines

- **Explicitness:** Use clear, descriptive names. Explicit types for public APIs.
- **Self-explanatory code:** Code must communicate intent; avoid comments.
- **Simplicity:** Apply SOLID, YAGNI, and KISS principles.
- **Use async/await over callbacks.**
- **Avoid magic numbers/strings; define constants.**
- **Use ESLint/Prettier, React/TypeScript plugins.**

## Type Safety

- **No `any` type usage.**
- **Use strict TypeScript configuration.**
- **Type all component props and return values.**
- **Use type guards for runtime type checking.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/techlibs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
