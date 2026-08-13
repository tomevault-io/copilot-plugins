## tailwindcssv4-rules

> You are working on a modern web project using TailwindCSS V4+. Follow these rules to ensure consistent, performant, and maintainable code.


# TailwindCSS V4+ Development Rules

## Project Context

You are working on a modern web project using TailwindCSS V4+. Follow these rules to ensure consistent, performant, and maintainable code.

## TailwindCSS V4+ Specific Guidelines

### New CSS Engine & Features

- Utilize the new Oxide CSS engine benefits (faster builds, better tree-shaking)
- Use CSS imports instead of PostCSS when possible: `@import "tailwindcss"`
- Leverage improved container queries: `@container` and container-type utilities
- Use the new `@layer` syntax for custom utilities and components
- Take advantage of improved CSS nesting support

### Modern Utility Patterns

- Prefer logical properties when available: `ms-4` over `ml-4`, `ps-6` over `pl-6`
- Use new color utilities with better contrast ratios
- Leverage enhanced grid utilities with subgrid support where applicable
- Utilize improved aspect-ratio utilities: `aspect-[4/3]` syntax
- Use new text-wrap utilities: `text-wrap`, `text-nowrap`, `text-balance`

### Performance Optimizations

- Use `@apply` sparingly - prefer utility-first approach
- Leverage JIT mode effectively with arbitrary values: `w-[calc(100%-2rem)]`
- Use CSS custom properties for dynamic values: `bg-[--custom-color]`
- Minimize custom CSS - utilize Tailwind utilities whenever possible

## Code Organization

### Component Structure

- Group related utilities logically: layout → spacing → colors → typography → effects
- Use consistent order: responsive prefixes → state modifiers → utilities
- Example order: `md:flex md:items-center hover:bg-blue-500 transition-colors`

### Responsive Design

- Mobile-first approach: base styles first, then breakpoint prefixes
- Use consistent breakpoint strategy: `sm:` `md:` `lg:` `xl:` `2xl:`
- Consider container queries for component-level responsiveness
- Use `max-*` utilities sparingly, prefer min-width breakpoints

### State Management

- Use state modifiers effectively: `hover:` `focus:` `active:` `disabled:`
- Leverage group and peer utilities for complex interactions
- Use `data-*` attribute styling: `data-[state=open]:opacity-100`
- Consider `has-*` pseudo-class utilities for parent state

## Best Practices

### Naming and Organization

- Use semantic class groupings in components
- Create custom utilities in `@layer utilities` when needed
- Use CSS custom properties for theme values
- Organize utilities by visual hierarchy importance

### Accessibility

- Include focus states for all interactive elements
- Use semantic color names and sufficient contrast ratios
- Implement proper focus management with `focus-visible:`
- Use `sr-only` for screen reader text when needed

### Performance

- Avoid deep nesting of arbitrary values
- Use shorthand utilities when available: `p-4` over `px-4 py-4` when equal
- Leverage Tailwind's built-in purging - avoid dynamic class construction
- Use CSS custom properties for values that change programmatically

### Dark Mode & Theming

- Implement consistent dark mode patterns: `dark:bg-gray-900`
- Use CSS custom properties for complex theme switching
- Consider `@media (prefers-color-scheme)` alongside class-based toggling
- Ensure all interactive states work in both light and dark modes

## Specific Patterns

### Layout Patterns

```html
<!-- Modern Grid -->
<div class="grid grid-cols-[repeat(auto-fit,minmax(300px,1fr))] gap-6">
  <!-- Flexible Container -->
  <div class="container mx-auto px-4 sm:px-6 lg:px-8">
    <!-- Sticky Header -->
    <header class="sticky top-0 z-50 bg-white/80 backdrop-blur-sm"></header>
  </div>
</div>
```

### Interactive Elements

```html
<!-- Button with all states -->
<button
  class="px-4 py-2 bg-blue-600 text-white rounded-lg 
               hover:bg-blue-700 focus:outline-none focus:ring-2 
               focus:ring-blue-500 focus:ring-offset-2 
               disabled:opacity-50 disabled:cursor-not-allowed
               transition-colors duration-200">
  <!-- Card with hover effects -->
  <div
    class="p-6 bg-white rounded-xl shadow-sm 
           hover:shadow-lg hover:-translate-y-1 
           transition-all duration-300"></div>
</button>
```

### Form Elements

```html
<!-- Input with focus states -->
<input
  class="w-full px-3 py-2 border border-gray-300 rounded-md
             focus:outline-none focus:ring-2 focus:ring-blue-500 
             focus:border-blue-500 invalid:border-red-500" />
```

## Code Quality Rules

### Consistency

- Always use the same spacing unit system (prefer rem-based: `space-4`)
- Maintain consistent component patterns across the project
- Use the same transition durations and easings throughout
- Follow the same color palette and naming conventions

### Maintainability

- Document complex utility combinations with comments
- Extract repeated patterns into components or CSS classes
- Use meaningful variable names for CSS custom properties
- Keep utility classes readable with proper line breaks in long lists

### Error Prevention

- Validate color contrast ratios meet WCAG standards
- Test all interactive states (hover, focus, active, disabled)
- Ensure responsive breakpoints work across devices
- Check that animations respect `prefers-reduced-motion`

## File Organization

- Import Tailwind at the top of your main CSS file
- Group custom utilities logically in separate files
- Use consistent naming for component stylesheets
- Keep configuration files clean and well-documented

## Integration Guidelines

- Configure content paths properly for optimal purging
- Use official plugins when available over custom solutions
- Integrate with build tools effectively (Vite, Webpack, etc.)
- Consider using Tailwind IntelliSense extensions

## Testing Considerations

- Test components across all breakpoints
- Verify accessibility with screen readers
- Check performance impact of custom utilities
- Validate that purging works correctly in production builds

Remember: The goal is to write maintainable, performant, and accessible code that leverages TailwindCSS V4+'s full potential while maintaining consistency across your project.

---
> Source: [ronnypdev/Notes-Web](https://github.com/ronnypdev/Notes-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
