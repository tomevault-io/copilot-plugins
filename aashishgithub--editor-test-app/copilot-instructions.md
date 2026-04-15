## editor-test-app

> You are assisting a Senior Frontend Cloud Architect with 12+ years of experience.

You are assisting a Senior Frontend Cloud Architect with 12+ years of experience.

EXPERTISE LEVEL:
- Expert-level React 18+ with advanced patterns (Suspense, Concurrent Features, Server Components)
- Advanced TypeScript with complex type systems and generics
- Cloud architecture patterns (AWS/GCP/Azure)
- Micro-frontends and module federation
- Performance optimization and Core Web Vitals
- Modern build tools (Vite, esbuild, Turbopack)
- Testing strategies (Jest, Vitest, Playwright, Cypress)

CODE & ARCHITECTURE STANDARDS:
- Use TypeScript strict mode throughout the codebase
- Follow SOLID principles, clean architecture, and separation of concerns
- Use container/presentational component patterns; separate business logic (hooks/services) from UI rendering
- Organize components, hooks, and services in feature-based directories (see PR_REVIEW_PROMPTS.md for structure)
- Name files and symbols according to conventions: Container (e.g., AppServiceInfoPanel), Presentation (e.g., AppEndpoint), Hooks (e.g., useAppService), Types (e.g., AppServiceProps)
- Design components and hooks for reusability and composability; prefer generic, well-typed props
- Place business logic in service layers and custom hooks; avoid logic in UI components
- Use dependency injection and composition over inheritance
- Optimize for tree-shaking, code splitting, and performance (memoization, minimal re-renders)
- Use React Query for server state, Context API for global state, and local state only where appropriate
- Ensure all dependencies are listed in useEffect /useMemo/useCallback arrays
- Write self-documenting code with JSDoc for all public APIs and complex logic
- Avoid 'any' types; use strict, reusable types and type guards
- Only export types if they are used outside the file; otherwise, keep types local to the file
- All new features must have unit and integration tests; tests should be isolated, meaningful, and follow codebase structure
- Reference and follow codebase examples for patterns and best practices
- New component files must be camelCased (e.g., MyComponent.tsx)
- Do not create many components unless they are designed for reuse or composability
- Avoid creating interfaces; prefer using Types for type definitions
- Use function declarations for all component definitions (e.g., export function MyComponent({ ... }))
- Do not use arrow functions for component definitions
- Do not use React.FC with type annotations for presentational components
- All function components must be defined using function declarations (see eslint rule: react/function-component-definition)
- All new code must follow the ESLint rules set for the project
- Default to creating generic, reusable, and composable components for similar UI/logic (e.g., DocConfigInput for doc config inputs) instead of duplicating code across multiple components. Prioritize DRY, composability, and consistency.
- Use typography components from 'components/typography' (Heading, Text) instead of native HTML elements (<h1>, <h2>, <p>, text in <div>, etc.)
- For all form implementations, see the "Form Usage Patterns" section below.

# Import Ordering
- ALWAYS follow the ESLint import ordering rules. The project uses automatic import sorting.
- Standard import order (enforced by ESLint):
  1. External libraries (react, react-dom, third-party packages)
  2. @couchbasecloud/* packages (e.g., @couchbasecloud/base-ui)
  3. Local component imports (components/*)
  4. Local page/feature imports (pages/*, partials/*)
  5. Type imports (types/*, type imports with 'type' keyword)
  6. Constants imports (constants/*)
  7. Utility imports (utils/*, hooks/*)
  8. Relative imports (./*, ../)
- Example of correct import order:
  ```typescript
  import { useState } from 'react';
  import { TextField } from '@couchbasecloud/base-ui/components/inputs/text-field';
  import { Button } from 'components/button/button';
  import { Text } from 'components/typography/text';
  import type { MyProps } from 'pages/feature/types';
  import { CONSTANTS } from 'constants/app';
  import makeDataAutoId from 'utils/make-data-auto-id';
  import { helperFunction } from './helper';
  ```
- If ESLint shows "Run autofix to sort these imports!", ALWAYS fix the import order immediately
- Never ignore import ordering warnings - they exist to maintain consistency across the codebase
- Use `type` keyword for type-only imports to enable better tree-shaking: `import type { MyType } from './types'`

# Design System Component Usage
- ALWAYS use design system components instead of native HTML elements
- Use `Button` component for all interactive buttons instead of native <button> elements
- Use `Heading` component for all headings instead of native HTML elements (<h1>, <h2>, etc.)
- Use `Text` component for all text content instead of native HTML elements (<p>, <span>, text in <div>, etc.)
- Use design system components for forms (Input, Select, Checkbox, etc.) instead of native form elements
- Follow the established typography scale and weight conventions:

## Button Component
- **Variants**: primary, secondary, tertiary, tertiary-text, danger, success
- **Usage**: `<Button variant="tertiary" onClick={handleClick}>Click Me</Button>`
- **Props**: variant (required), onClick, disabled, tooltip, dataAutoId, className, children, contentClassName
- Use `tertiary` variant for text-like clickable elements that need button functionality
- Use `tertiary-text` variant for text buttons with minimal styling
- **For simple text buttons**: Use `contentClassName` for typography styling instead of nesting `<Text>` components
  ```tsx
  // ✅ GOOD - Direct content styling
  <Button variant="tertiary" contentClassName="text-base font-medium text-on-background-link">
    Click Me
  </Button>

  // ❌ AVOID - Unnecessary nesting for simple text
  <Button variant="tertiary">
    <Text variant="t2" weight="medium">Click Me</Text>
  </Button>
  ```

### Icon-Only Buttons
- **NEVER wrap an Icon component as a child of Button**
- Instead, use the `iconOnly` prop with the `icon` prop
- **Icon-Only Variants**: secondary, secondary-error, copy, surface, success, warning, etc.
- **Required props**: `iconOnly={true}`, `icon="icon-name"`, `label="Accessible Label"`
- **Example**:
  ```tsx
  // ❌ BAD - Don't wrap Icon
  <Button variant="tertiary-text" onClick={handleClick}>
    <Icon name="delete" />
  </Button>

  // ✅ GOOD - Use iconOnly pattern
  <Button
    variant="secondary-error"
    iconOnly
    icon="delete"
    label="Delete"
    onClick={handleClick}
    tooltip="Delete"
    dataAutoId={dataAutoId}
  />
  ```

## Why Design System Components?
- Consistent styling and behavior across the application
- Built-in accessibility features (ARIA attributes, keyboard navigation)
- Proper focus management and states (hover, active, disabled)
- Standardized spacing and layout
- Type-safe props and better developer experience
- Centralized maintenance and updates

## Heading Component
- **Variants**: h1 (24px, semibold), h2 (20px, semibold), h3 (18px, semibold), h4 (16px, medium), h5 (14px, medium), h6 (12px, medium)
- **Usage**: `<Heading variant="h2">Page Title</Heading>`
- **Props**: variant (required), color (optional), children, className, dataAutoId
- **Color**: Pass color via the `color` prop without the `text-` prefix (e.g., `color="on-surface"`, `color="on-background-alternate"`)

## Text Component
- **Variants**: t1 (18px, medium), t2 (16px, medium), t3 (14px, regular), t4 (12px, regular), t5 (10px, regular), t6 (8px, regular)
- **Weights**: medium, regular, light
- **Usage**: `<Text variant="t2" weight="regular">Description text</Text>`
- **Props**: variant (required), weight (optional), color (optional), children, className, dataAutoId, italic (optional)
- **Color**: Pass color via the `color` prop without the `text-` prefix (e.g., `color="on-surface"`, `color="on-background-alternate"`)

## Typography Best Practices
- Choose appropriate variant based on semantic hierarchy, not visual appearance
- Use consistent weight patterns: semibold for headings, medium for emphasis, regular for body text
- **ALWAYS use the `color` prop for text colors, NEVER add `text-*` classes to `className`**
- Avoid mixing typography-related classes (font-, text-, tracking-, leading-, italic) in className as they will be overridden
- Use `dataAutoId` for testing and accessibility
- Prefer semantic variants over custom styling (e.g., use `t3` instead of `t2` with custom font-size)

### Examples:
**BAD** ❌
```tsx
<Text variant="t3" weight="medium" className="uppercase tracking-wide text-on-background-alternate">
<Heading variant="h2" className="text-on-surface">
```

**GOOD** ✅
```tsx
<Text variant="t3" weight="medium" color="on-background-alternate" className="uppercase tracking-wide">
<Heading variant="h2" color="on-surface">
```

AVOID:
- Basic explanations of fundamental concepts
- Overly verbose comments for obvious code
- Outdated patterns or deprecated APIs
- Suggestions below senior-level expertise
- Mixing business logic with UI rendering
- Unnecessary context providers or global state
- Unstructured or inconsistent file organization

REVIEW FOCUS AREAS (for PRs):
- Component structure, separation, and organization
- Hook implementation, dependencies, and performance
- State management approach (local, global, server)
- TypeScript usage and type safety
- Test coverage and quality
- Adherence to codebase patterns and conventions

For detailed review prompts and examples, see PR_REVIEW_PROMPTS.md.

# Form Usage Patterns
- Use [React Hook Form](https://react-hook-form.com/) for all non-trivial forms, especially those with dynamic fields, validation, or complex state.
- Always define a strict TypeScript type for the form data and use it with `useForm`, `useFormContext`, and `useFieldArray`.
- Use `useFormContext` in deeply nested components to access form state and methods, avoiding prop drilling.
- Use `useFieldArray` for dynamic lists of fields (e.g., add/remove bindings, items, etc.).
- All form mutations (add, remove, update) must be performed through the React Hook Form API, not local state.
- Validation logic should be colocated with the form definition, not scattered in UI components.
- Do not mix local state and form state for the same data.
- Ensure all form fields have proper labels and accessibility attributes.
- Write unit and integration tests for forms, covering validation, dynamic fields, and submission.

# Reference Examples for Form Usage
For best practices in form implementation, see:
- src/pages/product-hub/ai-functions/add/add-ai-functions.tsx
- src/components/settings-drawer/settings-drawer-form.tsx
- src/pages/database/datatools/udf/udf-drawer/udf-drawer-form.tsx
- src/components/eventing/eventing-bindings/eventing-bindings.tsx
- src/components/transformations/transformations.tsx
- src/components/workflow-source-fields/workflow-source-fields.tsx
- src/pages/columnar/workbench/components/query-settings/query-settings-form-fields.tsx
- src/components/provide-feedback-modal/provide-feedback-modal.tsx
- src/components/import-rules-dialog/import-rules-dialog.tsx
- src/components/configure-search-field-new/configure-search-field-new.tsx

These files demonstrate:
- Strict TypeScript typing for form data.
- Use of useForm, useFormContext, and useFieldArray.
- All mutations via React Hook Form API.
- Presentational components are stateless and only receive necessary props.
- Validation logic colocated with form definition.
- No mixing of local and form state.
- Accessibility and testability.

# Memoization Hooks Usage
- Do NOT use useCallback or useMemo in new code unless there is a clear, measured performance bottleneck. Memoization must be justified with evidence (e.g., profiling, re-render analysis). Default to inlining functions and values. Remove unnecessary useCallback/useMemo from all new and existing code.

# Hooks vs Utility Functions
- Do NOT write a React hook unless it is not possible to use a pure utility function. Hooks should only be used for logic that requires React state, context, or lifecycle. Prefer pure utility functions for stateless, reusable, or business logic.

# Hooks and Effects Usage
- Avoid creating custom hooks for business logic or state synchronization unless absolutely necessary. Prefer colocating logic in the parent component using state, props, and direct function calls.
- Avoid useEffect for business logic or state synchronization when the same can be achieved with explicit state, props, or event handlers. Only use useEffect for true React lifecycle needs (e.g., subscriptions, timers, external effects).
- Prefer explicit, testable, and predictable state/logic flow in the component tree over implicit effects or hooks abstraction.

# Module Organization & Barrel Exports
- Create index.ts files for feature directories to provide single entry points (barrel export pattern)
- Use barrel exports to control API surface and enable better tree-shaking
- Structure: `export * from './module-name'` for full exports, `export { specific } from './module'` for selective exports
- Barrel exports improve import ergonomics: `import { util1, util2 } from 'utils/feature'` vs multiple imports
- Only export what's needed publicly; keep internal utilities private

# Feature-Based Directory Structure
- Organize by feature/domain rather than file type within feature directories
- Structure: feature-name/
  ├── components/          # Feature-specific components
  ├── hooks/              # Feature-specific hooks
  ├── utils/              # Feature-specific utilities
  ├── types/              # Feature-specific types
  ├── constants/          # Feature-specific constants
  ├── feature-name.tsx    # Main feature component
  └── index.ts           # Barrel export
- Keep related functionality colocated within feature boundaries
- Use descriptive file names that indicate purpose (e.g., user-permissions.utils.ts)

# Permission-Driven Development
- Centralize permission checking logic in dedicated utility functions
- Build UI components that adapt to user permissions dynamically
- Provide clear, actionable error messages when permissions are insufficient
- Test permission scenarios thoroughly in unit tests
- Use permission objects to control feature access: `{ canCreate, canEdit, canDelete, canView }`
- Implement permission-based conditional rendering and feature flags

# Type Guards & Runtime Type Checking
- Create type guard functions for union types and complex data structures
- Use type guards for runtime type checking: `isProxyVectorTarget(target): target is ProxyVectorTarget`
- Implement type guards for API response validation
- Use type guards in conditional logic to provide TypeScript type narrowing
- Example pattern: `if (isVectorIndex(item)) { /* TypeScript knows item is VectorIndex */ }`

# Error Resilience & User Experience
- Implement automatic retry logic for transient failures (network, temporary server issues)
- Provide fallback UI states for error scenarios
- Handle edge cases gracefully (empty data, network failures, permission changes)
- Use error boundaries for component-level error handling
- Log errors appropriately for debugging while providing user-friendly messages
- Implement optimistic updates where appropriate with rollback on failure

# Smart Data Management
- Implement conditional polling based on data state (e.g., only poll when items are loading)
- Use pagination for large datasets with proper loading states
- Implement automatic retry logic for transient failures with exponential backoff
- Handle loading, error, and empty states consistently across components
- Use React Query for server state with appropriate stale times and cache invalidation

# Comprehensive Testing Strategy
- Write tests for custom hooks using @testing-library/react-hooks
- Test utility functions in isolation with comprehensive edge cases
- Test permission logic thoroughly with different user roles and scenarios
- Test form validation and dynamic field behavior
- Test error handling and retry logic
- Test component integration with proper mocking of dependencies
- Follow naming: feature-name.test.tsx, feature-name.utils.test.ts, use-feature-name.test.tsx

# Performance Optimization
- Use React.memo for expensive presentational components only when proven beneficial
- Implement proper dependency arrays in useEffect, useMemo, useCallback
- Use pagination and virtualization for large data sets
- Implement code splitting at route level and for heavy components
- Monitor and optimize bundle size with proper tree-shaking
- Use React Query's built-in caching and background updates efficiently

# Data Auto ID Pattern (Testing & Accessibility)
- All components must accept `dataAutoId?: string` as an optional prop (NOT a function)
- Components must use `makeDataAutoId` utility to create a function from the string prop
- Pattern implementation:
  ```typescript
  import makeDataAutoId from 'utils/make-data-auto-id';

  export function MyComponent({ dataAutoId, ...otherProps }: MyComponentProps) {
    const maybeDataAutoId = makeDataAutoId(dataAutoId, 'my-component');

    return (
      <div>
        <Button dataAutoId={maybeDataAutoId('submit')}>Submit</Button>
        <Input dataAutoId={maybeDataAutoId('email-input')} />
      </div>
    );
  }
  ```
- Type definition must use `dataAutoId?: string` (not a function type)
- This pattern enables test automation and accessibility tooling
- No deviation from this pattern is allowed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aashishGitHub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
