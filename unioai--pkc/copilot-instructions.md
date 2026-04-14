## pkc

> **Always use Shadcn UI components when available instead of custom implementations.**


# Shadcn UI Usage Rules

## Core Principle

**Always use Shadcn UI components when available instead of custom implementations.**

This document establishes guidelines for using Shadcn UI components consistently across our project to ensure simplicity, maintainability, and design coherence.

## Component Selection Rules

1. **Priority Order for UI Components:**
   - Shadcn UI component (first choice)
   - Extending a Shadcn UI component (second choice)
   - Custom component (last resort, only when no suitable Shadcn alternative exists)

2. **Never Duplicate Functionality:**
   - If Shadcn provides a component that matches our needs, use it even if a custom version already exists
   - Phase out existing custom components that duplicate Shadcn functionality

## Implementation Guidelines

1. **Component Styling:**
   - Use Shadcn's styling approach (Tailwind + class-variance-authority)
   - Follow Shadcn's theming variables for colors, spacing, and typography
   - Extend using the `cn()` utility function from `@/lib/utils` for class composition

2. **Component Architecture:**
   - Keep JSX markup as simple as possible by leveraging Shadcn components
   - Use Shadcn's composition patterns (e.g., Card.Header, Card.Content, Card.Footer)
   - Maintain component hierarchies as per Shadcn documentation

3. **Refactoring Guidance:**
   - Identify code with extensive CSS/styling and replace with equivalent Shadcn components
   - Replace multiple nested divs with Shadcn layout components
   - Convert custom form elements to Shadcn form components

## Examples of Simplification

### Before (Custom Implementation)
```astro
<div class="custom-card">
  <div class="custom-card-header">
    <h3 class="custom-card-title">Title</h3>
  </div>
  <div class="custom-card-body">
    <p class="custom-card-content">Content goes here</p>
  </div>
  <div class="custom-card-footer">
    <button class="custom-button">Action</button>
  </div>
</div>

<style>
  .custom-card {
    background-color: var(--color-card-bg);
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    /* More custom styles... */
  }
  /* Many more style definitions... */
</style>
```

### After (Shadcn Implementation)
```astro
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>
    <p>Content goes here</p>
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

## Benefits

1. **Simpler Code:** Reduced line count and complexity
2. **Consistency:** Unified design language across the application
3. **Maintainability:** Easier updates through centralized component system
4. **Performance:** Optimized components with better defaults
5. **Accessibility:** Built-in accessibility features from Shadcn

## Component Usage Checklist

Before implementing any UI element, check this order:
1. ✓ Does Shadcn provide this component already?
2. ✓ Can we compose existing Shadcn components to achieve this?
3. ✓ Can we extend a Shadcn component for our needs?
4. ✓ Only then consider a custom implementation

## Migration Strategy

1. Identify high-impact pages for initial conversion
2. Start with container/layout components
3. Replace custom components with Shadcn alternatives
4. Update theming variables to match Shadcn's system
5. Validate functionality and appearance after each component migration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unioai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
