## metrolink-mcp

> These rules provides guidance for React 19, always follow these rules when writing React 19 code


# React 19 Style Guide

## Component Patterns

### TypeScript & Component Definition

- **Arrow functions only** for all components (no function declarations)
- Components with props must:
  1. Define a Props interface named `ComponentNameProps` (e.g., `ButtonProps` for `Button`)
  2. Use React's `FC` generic, explicitly imported: `import type { FC } from 'react'`
  3. Apply Props interface: `const Component: FC<ComponentProps> = ({ prop1, prop2 }) => (...)`

### Async Components

- Use async arrow functions: `const MyComponent = async () => {...}`
- Apply to components when using React 19 async capabilities or when used in Server Component contexts

### Regular Components

- Follow the same arrow function + `FC<Props>` pattern
- Props interface named `ComponentNameProps`
- Example:
  ```tsx
  import type { FC } from 'react';
  
  interface ButtonProps {
    label: string;
    onClick: () => void;
  }
  
  const Button: FC<ButtonProps> = ({ label, onClick }) => (
    <button onClick={onClick}>{label}</button>
  );
  
  export default Button;
  ```

## Import Organization

- Group imports in this order:
  1. React core imports
  2. Third-party library imports
  3. Internal component imports
  4. Type-only imports (using `import type`)
  5. Relative imports (utilities, hooks, types)

- Use `import type` for type-only imports
- Organize imports alphabetically within each group

## CSS Class Management with clsx

### Basic Rules

- **Single class**: Use directly in `className` without `clsx`
  ```tsx
  <div className="container">...</div>
  ```

### Multiple Classes (5 or fewer)

- For 5 or fewer classes, use string concatenation or template literals
  ```tsx
  <div className="container mx-auto px-4 py-2">...</div>
  ```

### Multiple Classes (more than 5)

- Use `clsx` with **array notation**, grouped by type where possible
- Maximum of 5 classes per line (can be within the same string)
- Group related classes together (e.g., layout, spacing, colors, typography)
- Example:
  ```tsx
  import clsx from 'clsx';
  
  <div
    className={clsx(
      // Layout
      'container', 'mx-auto', 'flex', 'flex-col', 'items-center',
      // Spacing
      'px-4', 'py-2', 'gap-4', 'mb-6',
      // Colors & Background
      'bg-white', 'text-gray-900', 'border', 'border-gray-200',
      // Typography
      'text-lg', 'font-semibold', 'leading-relaxed'
    )}
  >
    ...
  </div>
  ```

### Conditional Classes

- **ONLY use Object notation form** for conditional classes
- Example:
  ```tsx
  import clsx from 'clsx';
  
  <div
    className={clsx(
      'base-class',
      {
        'active-class': isActive,
        'disabled-class': isDisabled,
        'hover-class': isHovered,
      }
    )}
  >
    ...
  </div>
  ```

### Combining classNames Prop

- When a component accepts a `classNames` prop, pass it as the second parameter to `clsx`
- Example:
  ```tsx
  interface ButtonProps {
    classNames?: string;
    // ... other props
  }
  
  const Button: FC<ButtonProps> = ({ classNames, ...props }) => (
    <button
      className={clsx(
        'btn', 'btn-primary', 'px-4', 'py-2',
        classNames
      )}
      {...props}
    />
  );
  ```

### Mixed Usage (Conditional + Multiple Classes)

- Combine array notation for base classes with object notation for conditionals
- Example:
  ```tsx
  <div
    className={clsx(
      // Base classes (array notation)
      'container', 'mx-auto', 'px-4', 'py-2',
      // Conditional classes (object notation)
      {
        'bg-white': theme === 'light',
        'bg-gray-900': theme === 'dark',
        'hidden': !isVisible,
      },
      // Additional classNames prop
      classNames
    )}
  >
    ...
  </div>
  ```

## Custom Hooks

- File naming: `useHookName.ts` or `useHookName.tsx` (if it returns JSX)
- Hook naming: `use` prefix (e.g., `useAuth`, `useDataFetch`)
- Return tuple for multiple values: `[value, setValue]` or `{ value, setValue }` object
- Place hooks in `src/hooks/` directory

## File Naming Conventions

- Components: PascalCase (e.g., `UserProfile.tsx`)
- Hooks: camelCase with `use` prefix (e.g., `useAuth.ts`)
- Utilities: camelCase (e.g., `formatDate.ts`)
- Types: PascalCase (e.g., `UserTypes.ts`)
- Constants: UPPER_SNAKE_CASE (e.g., `API_ENDPOINTS.ts`)

## Error Handling

### Component-Level Error Handling

- Handle errors at the component level using try-catch blocks and error states
- Use React error boundaries only when necessary for specific component trees
- Pattern for component-level error handling:
  ```tsx
  import type { FC } from 'react';
  import { useState } from 'react';
  
  interface ComponentProps {
    // props
  }
  
  const Component: FC<ComponentProps> = ({ ...props }) => {
    const [error, setError] = useState<Error | null>(null);
  
    const handleAction = async () => {
      try {
        // async operation
      } catch (err) {
        setError(err instanceof Error ? err : new Error('Unknown error'));
      }
    };
  
    if (error) {
      return <div>Error: {error.message}</div>;
    }
  
    return (
      // component JSX
    );
  };
  
  export default Component;
  ```

## Suspense Boundaries

### Component-Level Suspense

- Use Suspense for individual async components within a page
- Allows partial page loading and streaming
- Pattern:
  ```tsx
  const Container = () => {
    return (
      <div>
        <Header />
        <Suspense fallback={<Skeleton />}>
          <AsyncComponent />
        </Suspense>
      </div>
    );
  };
  ```

### Fallback Component Patterns

- Create reusable fallback components (skeletons, spinners, etc.)
- Fallback components should match the layout of the loading content
- Use skeleton components for better UX
- Example:
  ```tsx
  // Skeleton component
  const UserProfileSkeleton: FC = () => (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/4 mb-4"></div>
      <div className="h-4 bg-gray-200 rounded w-1/2"></div>
    </div>
  );
  
  // Usage
  <Suspense fallback={<UserProfileSkeleton />}>
    <UserProfile />
  </Suspense>
  ```

## Accessibility

### Semantic HTML

- **Always use semantic HTML elements** instead of generic divs/spans
- Use appropriate elements: `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`
- Use heading hierarchy correctly (`h1` → `h2` → `h3`, etc.)
- Use `<button>` for interactive elements, not `<div>` with onClick
- Use `<label>` for form inputs
- Example:
  ```tsx
  // ✅ CORRECT
  <main>
    <article>
      <h1>Article Title</h1>
      <p>Article content</p>
    </article>
  </main>
  
  // ❌ WRONG
  <div>
    <div>
      <div>Article Title</div>
      <div>Article content</div>
    </div>
  </div>
  ```

### ARIA Attributes

- Use ARIA attributes when semantic HTML is not sufficient
- Use `aria-label` for icon-only buttons
- Use `aria-describedby` to associate descriptions with form inputs
- Use `aria-live` for dynamic content updates
- Use `role` attribute only when necessary (prefer semantic HTML)
- Example:
  ```tsx
  <button aria-label="Close dialog">
    <CloseIcon />
  </button>
  
  <input
    aria-describedby="email-error"
    aria-invalid={hasError}
  />
  {hasError && <span id="email-error">Invalid email</span>}
  ```

### Keyboard Navigation

- Ensure all interactive elements are keyboard accessible
- Use proper tab order (logical sequence)
- Provide visible focus indicators
- Handle Enter and Space keys for buttons
- Handle Escape key for modals and dialogs
- Example:
  ```tsx
  const Button: FC<ButtonProps> = ({ onClick, children }) => {
    const handleKeyDown = (e: React.KeyboardEvent) => {
      if (e.key === 'Enter' || e.key === ' ') {
        e.preventDefault();
        onClick();
      }
    };
  
    return (
      <button
        onClick={onClick}
        onKeyDown={handleKeyDown}
        className="focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        {children}
      </button>
    );
  };
  ```

### Focus Management

- Manage focus for modals, dialogs, and dynamic content
- Trap focus within modals
- Return focus to trigger element when modal closes
- Set initial focus on modal open
- Example:
  ```tsx
  const Modal: FC<ModalProps> = ({ isOpen, onClose, children }) => {
    const modalRef = useRef<HTMLDivElement>(null);
  
    useEffect(() => {
      if (isOpen && modalRef.current) {
        // Focus first focusable element
        const firstFocusable = modalRef.current.querySelector(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        ) as HTMLElement;
        firstFocusable?.focus();
      }
    }, [isOpen]);
  
    if (!isOpen) return null;
  
    return (
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        className="focus:outline-none"
      >
        {children}
      </div>
    );
  };
  ```

## Code Organization

### Barrel Exports (index.ts)

- Use barrel exports (`index.ts`) to simplify imports
- Place `index.ts` in directories to re-export public APIs
- Only export what should be public
- Pattern:
  ```ts
  // src/components/index.ts
  export { Button } from './Button';
  export { Input } from './Input';
  export { Card } from './Card';
  
  // Usage
  import { Button, Input, Card } from '@/components';
  ```

### Module Boundaries

- Keep clear boundaries between modules
- Avoid circular dependencies
- Use absolute imports with path aliases (`@/components`, `@/lib`, etc.)
- Keep shared code in `src/lib/` or `src/shared/`

## Additional Patterns

- Prefer explicit type annotations over inference for component props
- Keep components focused and single-purpose
- Extract complex logic into custom hooks or utilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimdrury)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jimdrury)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
