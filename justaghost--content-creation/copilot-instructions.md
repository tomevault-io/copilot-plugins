## content-creation

> **Scope:** `app/(dashboard)/**`, `app/(marketing)/**`, `components/**`, `hooks/**`, `styles/**`

# Frontend Team Rules

**Scope:** `app/(dashboard)/**`, `app/(marketing)/**`, `components/**`, `hooks/**`, `styles/**`

Dashboard UI, marketing pages, shared components, hooks, and styling.

## Component Standards

- ALWAYS use functional components with hooks. No class components.
- PREFER named exports over default exports.
- ALWAYS define prop types with TypeScript interfaces.
- USE CSS Modules for component-specific styles (`Component.module.css`).
- Extract complex logic into custom hooks in `hooks/`.

## File Naming

- Components: PascalCase (`ContentCard.tsx`)
- CSS Modules: Match component (`ContentCard.module.css`)
- Hooks: camelCase with `use` prefix (`useContentList.ts`)

## Accessibility (WCAG 2.1 AA)

- Use semantic HTML elements (`button`, `nav`, `main`, not `div` with onClick).
- Add ARIA labels to all interactive elements.
- Ensure keyboard navigation works for all interactions.
- Maintain color contrast ratio of 4.5:1 minimum.

## Performance

- Use `React.memo()` for expensive components.
- Use `useMemo()` and `useCallback()` to prevent unnecessary re-renders.
- Lazy load components with `dynamic()` imports when appropriate.
- Use Next.js `Image` component for optimized images.

## TypeScript

- Strict mode enforced. No `any` types.
- Define interfaces for all component props.

## Component Pattern

```tsx
import styles from './MyComponent.module.css';

interface MyComponentProps {
  title: string;
  onAction: () => void;
}

export function MyComponent({ title, onAction }: MyComponentProps) {
  return (
    <div className={styles.container}>
      <h2>{title}</h2>
      <button onClick={onAction} aria-label="Perform action">
        Action
      </button>
    </div>
  );
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustAGhosT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
