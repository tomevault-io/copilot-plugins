## tenantweb-test

> description: PrimeVue component usage best practices for Vue 3 applications using PrimeVue library

---
description: PrimeVue component usage best practices for Vue 3 applications using PrimeVue library
globs: **/*.vue, **/*.ts, **/*.js
---

# PrimeVue Best Practices

## Component Usage
- Prefer PrimeVue components over native HTML elements (e.g., `Button` over `<button>`, `InputText` over `<input>`)
- Use correct PrimeVue props (e.g., `label`, `modelValue`, `options`, `severity`)
- Avoid unnecessary wrappers around PrimeVue components
- Always reference official [PrimeVue docs](mdc:https:/primevue.org) for component usage
- Use `v-model` for two-way data binding with inputs
- Use PrimeVue `Dialog`, `ConfirmDialog` for modals and notifications
- For Toast use [notification-service.ts](mdc:src/services/notification-service.ts) that implement the primevue Toast

## Layout
- Use `PrimeFlex` or TailwindCSS for layout utilities (not inline styles)
- Use `Grid`, `Flex`, `Card`, and `Panel` components from PrimeVue where applicable
- Leverage `p-fluid` class for fluid layouts
- Use spacing and margin classes (`p-mb-3`, `p-mt-2`) consistently

## Forms
- Use `InputText`, `Dropdown`, `Textarea`, `Calendar`, `InputNumber`, `Checkbox`, `RadioButton` from PrimeVue
- Use `FormKit` or `vee-validate` for advanced form management (optional)
- Always use labels and accessibility attributes
- Group form elements with `Fieldset` or `Panel`

## Data Display
- Use `DataTable` for tabular data with pagination, filters, and sorting
- Configure column templates and row actions clearly
- Use `Card`, `Accordion`, `TabView` for structured data
- Use `Skeleton` or `ProgressSpinner` for loading states

## Navigation
- Use `Menubar`, `TabMenu`, `TieredMenu`, or `Sidebar` components
- Configure navigation via `router-link` or programmatic routing
- Maintain consistent navigation patterns

## Accessibility
- Ensure all interactive components (buttons, inputs, dialogs) follow ARIA standards
- Use PrimeVue's accessible components like `Dialog` and `Toast`
- Avoid disabling focus outlines without fallback styling

## Theming & Styling
- Stick to a consistent PrimeVue theme across the app
- Customize theme variables if needed using SCSS or CSS variables
- Use `unstyled` props when custom styling is preferred
- Avoid inline styles, use class-based styling from PrimeVue or TailwindCSS

## Performance
- Import components on demand when not using global registration
- Avoid loading all PrimeVue components unnecessarily
- Use lightweight alternatives when component is too heavy for simple tasks
- Use lazy loading and suspense for large component blocks

## Best Practices
- Use PascalCase for component names in `<template>` (e.g., `<Button />`, `<InputText />`)
- Keep component imports organized alphabetically
- Comment non-trivial component usage
- Stick to single-responsibility principle for components
- Validate PrimeVue upgrades before pushing to production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omar1893) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
