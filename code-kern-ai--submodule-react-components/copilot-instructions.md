## submodule-react-components

> Guidelines for developing in the react-components submodule


# React Components Submodule

This is a shared component library. Changes here affect all consuming applications. The submodules are used in the following FE repositories: cognition-ui, refinery-ui, admin-dashboard and refinery-entry.

## Directory Structure

```
components/         # UI components
hooks/              # Custom React hooks
helpers/            # Component helper functions
types/              # TypeScript interfaces
assets/             # Static assets
```

## Component Guidelines

### Export Pattern

Use default exports for components:

```typescript
export default function MyComponent(props: MyComponentProps) { ... }
```

### Props Types

Define types in `types/` directory:

```typescript
// types/my-component.ts
export type MyComponentProps {
  required: string;
  optional?: number;
}
```

### Styling

- Use Tailwind CSS classes
- Use `combineClassNames` from javascript-functions for conditional classes
- Avoid inline styles

## Hook Guidelines

- Hooks must start with `use`
- File name should match hook name
- Document return types explicitly

## Dependencies

- `@headlessui/react` - Accessible UI primitives
- `@nextui-org/react` - Tooltip component
- `@tabler/icons-react` - Icon library

## Component-Specific Guidelines

### KernTable

See [kern-table.mdc](./kern-table.mdc) for comprehensive guidelines on using the KernTable component, including:
- Props structure and configuration
- Table data preparation patterns
- Sorting implementation
- State management with hooks
- Performance optimization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/code-kern-ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/code-kern-ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
