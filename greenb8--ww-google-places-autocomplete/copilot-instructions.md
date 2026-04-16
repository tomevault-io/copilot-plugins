## ww-google-places-autocomplete

> This document outlines the coding standards and best practices to be followed throughout the project. Consistency in our code is crucial for readability, maintainability, and collaboration.


# Coding Standards

This document outlines the coding standards and best practices to be followed throughout the project. Consistency in our code is crucial for readability, maintainability, and collaboration.

## 1. General Principles

- **DRY (Don't Repeat Yourself)**: Avoid duplicating code. Abstract and reuse components and functions wherever possible.
- **Readability**: Write code that is easy to understand. Prioritize clarity over cleverness. Use comments to explain complex logic.
- **Completeness**: All features must be fully implemented. Leave no `TODO` comments, placeholders, or incomplete functionality.

## 2. Naming Conventions

- **Variables and Functions**: Use descriptive, camelCase names (e.g., `primaryTextColor`, `calculateTotal`).
- **Event Handlers**: Prefix event handling functions with `handle` (e.g., `handleClick`, `handleInputChange`).
- **Components**: Component files and directories should use kebab-case (e.g., `my-custom-button`).

## 3. JavaScript/Vue.js

- **Function Definitions**: Use `const` with arrow functions for component methods and utilities (e.g., `const myFunc = () => { ... };`).
- **Vue Props**: All props received from WeWeb should be handled through the `content` object.
- **Early Returns**: Use early returns (guard clauses) to reduce nesting and improve readability.

## 4. Styling

- **TailwindCSS Only**: All styling must be done using TailwindCSS utility classes. Do not use `<style>` tags in components or external CSS files.
- **Conditional Classes**: When applying classes conditionally, prefer the `class:` syntax over ternary operators for better readability if the framework supports it. Otherwise, use clear and concise ternaries.

## 5. Accessibility (a11y)

- **Semantic HTML**: Use semantic HTML5 elements where appropriate.
- **Interactive Elements**: All interactive elements (e.g., clickable `div`s, custom buttons) must be fully accessible. This includes:
  - `tabindex="0"` to make them focusable.
  - A descriptive `aria-label`.
  - Keyboard event handlers (`on:keydown` or `@keydown`) to handle actions for keys like `Enter` and `Space`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/greenb8) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
