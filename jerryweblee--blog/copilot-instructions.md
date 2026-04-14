## blog

> This rule provides guidelines for server-side development in Nuxt 4, covering API routes, server middleware, database interactions, and server-side rendering best practices.



# Nuxt 4 Vue Component Standards

## Component Structure

- Use Composition API as the primary approach for component logic
- Organize component sections in this order: `<script setup>`, `<template>`, `<style>`
- Use TypeScript for better type safety and developer experience
- Implement proper prop validation with runtime checks

## Script Setup Best Practices

- Use `<script setup>` syntax for all components
- Define props using `defineProps()` with TypeScript interfaces
- Use `defineEmits()` for event handling with proper typing
- Implement `defineExpose()` when exposing component methods to parent
- Use composables for reusable logic extraction

## Template Guidelines

- Use semantic HTML elements for better accessibility
- Implement proper ARIA attributes for interactive elements
- Use Vue directives efficiently (`v-if`, `v-show`, `v-for`)
- Avoid complex expressions in templates - move logic to computed properties
- Use slot patterns for flexible component composition

## Styling Standards

- Use scoped styles by default (`<style scoped>`)
- Implement CSS custom properties for theming
- Use utility classes for common styling patterns
- Follow BEM methodology for complex component styling
- Implement responsive design with mobile-first approach

## Component Communication

- Use props for parent-to-child data flow
- Implement events for child-to-parent communication
- Use provide/inject for deep component communication
- Avoid direct prop mutation - use events instead
- Implement proper validation for all data inputs

## Performance Optimization

- Use `v-memo` for expensive template calculations
- Implement proper key attributes for `v-for` loops
- Use `shallowRef` and `shallowReactive` for large objects
- Implement lazy loading for heavy components
- Use `defineAsyncComponent` for code splitting

## Accessibility Standards

- Implement proper semantic HTML structure
- Use ARIA labels and descriptions where needed
- Ensure keyboard navigation support
- Maintain proper color contrast ratios
- Test with screen readers and assistive technologies

## Error Handling

- Implement error boundaries for component failures
- Use try-catch blocks for async operations
- Provide fallback UI for error states
- Log errors appropriately for debugging
- Handle loading states gracefully

## Testing Considerations

- Write unit tests for component logic
- Test component interactions and events
- Mock external dependencies properly
- Test accessibility features
- Implement visual regression testing

## Common Patterns

- Use renderless components for logic-only functionality
- Implement compound components for complex UI patterns
- Use teleport for modal and overlay components
- Implement proper form handling with validation
- Use transition components for smooth animations

## Anti-patterns to Avoid

- Avoid mutating props directly
- Don't use `v-if` and `v-for` on the same element
- Avoid complex template expressions
- Don't overuse global state for local component data
- Avoid creating overly complex components

## TypeScript Integration

- Define proper interfaces for component props
- Use generic types for flexible components
- Implement proper event typing with `defineEmits`
- Use utility types for common patterns
- Maintain strict type checking throughout

## Component Documentation

- Document component purpose and usage
- Provide examples for complex components
- Document prop types and validation rules
- Include accessibility considerations
- Maintain changelog for component updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JerryWebLee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JerryWebLee)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
