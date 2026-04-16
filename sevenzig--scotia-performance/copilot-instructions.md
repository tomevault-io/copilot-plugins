## scotia-performance

> This guide covers best practices for organizing and encapsulating styles in Svelte components to prevent style leakage and conflicts.

# Component Style Encapsulation

This guide covers best practices for organizing and encapsulating styles in Svelte components to prevent style leakage and conflicts.

## Core Principles

### Components Own Their Styles

1. **Styles belong in the component that uses them**
   - Each component should contain its own `<style>` block
   - Avoid placing component-specific styles in global files
   - No component should style another component's elements

2. **Scope all component styles**
   - Use Svelte's built-in style encapsulation (default behavior)
   - Never use `:global()` for component-specific styles
   - Add component class prefixes for additional isolation

```svelte
<!-- Good: Encapsulated component styles -->
<style>
  /* These styles only affect this component */
  .card {
    border-radius: 0.5rem;
  }
</style>
```

### Global Style Management

3. **Global styles belong in dedicated files**
   - Place global styles in `src/lib/styles/global.css` (or similar)
   - Import global styles only in `+layout.svelte` or app entry points
   - Limit global styles to design tokens, resets, and utility classes

4. **Avoid duplicating selectors across scopes**
   - Never create the same selector in both global and component scopes
   - When a conflict occurs, increase specificity carefully, not with `!important`

```css
/* Bad: Duplicating selectors across files */
/* In global.css */
.button { /* styles */ }

/* In Button.svelte */
.button { /* conflicting styles */ }
```

### Critical CSS Management

5. **Critical CSS handling**
   - For above-the-fold critical CSS, use `<svelte:head>` only in page-level components
   - Never put component styles in page `<svelte:head>` sections
   - If using critical CSS, ensure no selector conflicts with components

```svelte
<!-- Avoid this pattern in +page.svelte -->
<svelte:head>
  <style>
    /* Don't place component-specific styles here */
    .hero { /* styles that will conflict */ }
  </style>
</svelte:head>

<!-- Later using the component -->
<Hero />
```

### CSS Naming and Inheritance

6. **Use consistent naming conventions**
   - Adopt BEM, SMACSS, or another convention consistently
   - Add component prefixes to avoid cross-component clashes
   - Document your naming strategy in your project documentation

```svelte
<!-- Using BEM with component prefixes -->
<style>
  .my-button { /* Base */ }
  .my-button__icon { /* Element */ }
  .my-button--primary { /* Modifier */ }
</style>
```

7. **Composition over inheritance**
   - Prefer component composition over style inheritance
   - Use Svelte props and slots to customize components
   - Create utility components for shared styling patterns

## Hydration Considerations

8. **SSR Style Consistency**
   - Ensure styles applied during SSR match client-side styles
   - Test components in both SSR and CSR environments
   - Consider using style props instead of dynamic styles

9. **Avoid post-hydration style flashes**
   - Do not conditionally apply structural styles based on client-side logic
   - Use CSS variables for dynamic styling that needs to change after hydration
   - Test components with network throttling to catch style flash issues

## Common Anti-Patterns

- ❌ **Never** use `!important` to solve component style conflicts
- ❌ **Never** place component styles in page-level style blocks
- ❌ **Never** style components from parent components
- ❌ **Never** use inline styles for structural layout
- ❌ **Never** use global browser reset styles within components

## Examples

### Good Example: Properly Encapsulated Component

```svelte
<!-- Button.svelte -->
<script>
  export let variant = 'default';
  export let size = 'medium';
</script>

<button class="button" class:button--primary={variant === 'primary'} class:button--small={size === 'small'}>
  <slot />
</button>

<style>
  .button {
    /* Base button styles */
    border-radius: 4px;
    padding: 0.5rem 1rem;
  }
  
  .button--primary {
    background-color: var(--color-primary);
    color: white;
  }
  
  .button--small {
    padding: 0.25rem 0.5rem;
    font-size: 0.875rem;
  }
</style>
```

### Bad Example: Style Leakage and Conflicts

```svelte
<!-- Page.svelte -->
<svelte:head>
  <style>
    /* Bad: This will conflict with the Button component */
    .button {
      background-color: blue;
    }
  </style>
</svelte:head>

<Button />

<!-- Button.svelte -->
<script>
  // Button logic
</script>

<button class="button">Click me</button>

<style>
  /* Styles will conflict with those in the page head */
  .button {
    background-color: red;
  }
</style>
```

## Checklist for Component Styling

- [ ] All component styles are contained within their component file
- [ ] No `:global()` selectors are used for component-specific styles
- [ ] No component styles are placed in page `<svelte:head>` sections
- [ ] No style selectors are duplicated across different files
- [ ] BEM or another naming convention is used consistently
- [ ] No `!important` is used to solve style conflicts
- [ ] Styles are tested for SSR/CSR consistency and hydration flashes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sevenzig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
