## chaos-campaign

> You are an expert in TypeScript, Vite, Vue 3, Vue Router, Pinia, Vuetify, tanstack query, msw, and Tailwind, with a deep understanding of best practices and performance optimization techniques in these technologies.

You are an expert in TypeScript, Vite, Vue 3, Vue Router, Pinia, Vuetify, tanstack query, msw, and Tailwind, with a deep understanding of best practices and performance optimization techniques in these technologies.

Code Style and Structure

- Write concise, maintainable, and technically accurate TypeScript code with relevant examples.
- Use functional and declarative programming patterns; avoid classes.
- Favor iteration and modularization to adhere to DRY principles and avoid code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Organize files systematically: each file should contain only related content, such as exported components, subcomponents, helpers, static content, and types.
- Define components using vue's composition api
- Use script setup for single file components

Naming Conventions

- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Favor named exports for functions.

Testing

- Use vitest for testing.
- Use vue testing library for testing vue components.
- Use msw for mocking API requests.
- Use chancejs for generating mixins for custom types.
- Add test files to the same directory as the file they are testing.
- use .spec.ts suffix for test files.
- Never mock child components in component tests; test the actual component integration
- Focus on testing component behavior and user interactions rather than implementation details
- Follow Testing Library philosophy
- Avoid testing specific text content; test structure and behavior instead
- Only test dynamic behavior and state changes that could break during runtime
- Global testing pinia is configured in beforeEach hook in setup.ts
- Initialize any Pinia stores prior to calling vgmRender
- Mock and verify console functions (log, error, info, debug) when called in tested function
- Never test:
  - Component existence or mounting
  - Static layout structure or DOM elements
  - Hard-coded text content
  - Component props that don't change
  - Default Vuetify component behavior

Accessibility

- Adhere to WCAG 2.2 standards.
- Use Vuetify's built-in accessibility features.
  TypeScript Usage
- Use TypeScript for all code; prefer interfaces over types for their extendability and ability to merge.
- Avoid enums; use maps instead for better type safety and flexibility.
- Use functional components with TypeScript interfaces.

UI and Styling

- Use Vuetify and Tailwind for components and styling.
- Implement responsive design with Tailwind CSS; use a mobile-first approach.

Performance Optimization

- Wrap asynchronous components in Suspense with a fallback UI.
- Use dynamic loading for non-critical components.
- Optimize images: use WebP format, include size data, implement lazy loading.
- Implement an optimized chunking strategy during the Vite build process, such as code splitting, to generate smaller bundle sizes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnnycricket) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
