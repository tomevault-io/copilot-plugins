## tailwind-styling

> This rule outlines our Tailwind CSS styling conventions.


# Tailwind CSS Styling Practices

This rule outlines our Tailwind CSS styling conventions.

## Class Organization

Organize Tailwind classes in logical groups:

1. Layout/positioning classes first
2. Sizing classes
3. Spacing (margin/padding)
4. Visual styles (colors, borders)
5. Typography
6. Interactive states
7. Responsive variants last

Example:
```tsx
className="flex flex-col gap-4 w-full p-6 bg-primary-100/20 text-sm hover:bg-primary-200/30 md:flex-row"
```

## Responsive Design

- Mobile-first approach (base classes for mobile, prefixed classes for larger screens)
- Use responsive prefixes: `sm:`, `md:`, `lg:`, `xl:`, `2xl:`

## Color System

- Use semantic color naming with numeric scale (primary-100, primary-900)
- Apply opacity with slash notation: `bg-primary-100/20`
- Use consistent dark mode variants: `dark:bg-primary-900/10`

```tsx
className="bg-primary-100/20 text-primary-900 dark:bg-primary-900/10 dark:text-primary-100"
```

## Layout Patterns

- Use flex and grid for layouts
- Use gap utilities instead of margins between flex/grid children
- Container classes for width constraints: `container-narrow`, `container-wide`, `container-ultrawide`, `max-w-sm` etc.

## Design System Integration

- Use consistent color palette (primary, secondary)
- Use consistent spacing scale
- Apply opacity for subtle UI elements
- Use gradient backgrounds for visual interest

```tsx
className="bg-gradient-to-r from-gray-50/5 via-gray-100/60 to-gray-50/5"
```

## Styling Approach
- Shadcn UI for behavior + TailwindCSS for styling
- Use `cn()` utility for conditional classes
- Leverage CVA for component variants

## Best Practices
- Use composition and props to extend functionality.
- Follow accessibility and responsive design patterns as established in the codebase.
- Prefer functional, declarative usage and avoid class-based components.

---
> Source: [PageAI-Pro/vibe-coding-starter](https://github.com/PageAI-Pro/vibe-coding-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
