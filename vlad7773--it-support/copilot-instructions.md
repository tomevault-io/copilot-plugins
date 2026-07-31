## rulesduplom

> description: TypeScript and React development guidelines

---
description: TypeScript and React development guidelines
alwaysApply: false
---

# TypeScript and React Style Guide

## General Rules

Write concise, technical TypeScript. Follow Standard.js rules.

Use functional, declarative patterns; avoid classes.

Favor loops and small helper modules over duplicate code.

Use descriptive names with auxiliary verbs (e.g. isLoading, hasError).

## Component Structure

File layout: exported component → subcomponents → hooks/helpers → static content.

Define props with interfaces/types, not prop-types.

Use the function keyword for components:

```ts
interface ButtonProps {
  label: string
  onClick?: () => void
}

export function Button({ label, onClick }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>
}
```

## React Hooks

Call hooks (useState, useEffect, etc.) only at the top level.

Extract reusable logic into custom hooks (useAuth, useFormValidation).

Memoize with React.memo, useCallback, useMemo where appropriate.

Avoid inline functions in JSX—pull handlers out or wrap in useCallback.

## Best Practices

Favor composition (render props, children) over inheritance.

Use React.lazy + Suspense for code splitting.

Use refs only for direct DOM access.

Prefer controlled components for forms.

Implement an error boundary component.

Clean up effects in useEffect to prevent leaks.

Use guard clauses (early returns) for error handling.

## Styling

Co‑locate a .scss file with each component.

Leverage SCSS features:
- Variables ($primary-color, $spacing)
- Mixins (@mixin flexCenter)
- Parent selector & for pseudo‑classes (&:hover)
- Partials (_variables.scss, _mixins.scss) imported in styles/index.scss

Name classes in camelCase or BEM (.card__header).

Keep global styles minimal (e.g. reset, typography).

## Forms

Use controlled inputs.

For simple forms, write custom hooks; for complex ones, use react-hook-form with generics (e.g. <Controller>).

Separate client‑side and server‑side validation.



Optionally integrate a schema library (e.g. Joi) if needed

---
> Source: [Vlad7773/it_support](https://github.com/Vlad7773/it_support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
