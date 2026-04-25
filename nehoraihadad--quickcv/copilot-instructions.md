## quickcv

> This rule provides guidelines for TypeScript usage and styling conventions in the QuickCV project.

# TypeScript and Style Conventions

This rule provides guidelines for TypeScript usage and styling conventions in the QuickCV project.

## TypeScript
- Use TypeScript for all new code
- Define interfaces and types in [src/types](mdc:src/types)
- For component props, create interfaces named `[ComponentName]Props`
- Use appropriate TypeScript features:
  - Union types for values that can be one of several types
  - Intersection types for combining types
  - Generic types for reusable type patterns
  - Type guards for runtime type checking
- Avoid using `any` type; use `unknown` if the type is truly unknown
- Use readonly arrays and properties when appropriate
- Use TypeScript's utility types (Pick, Omit, Partial, etc.) when appropriate

## Type Definitions
- For complex objects, define interfaces 
- For simple value sets, use union types or enums
- For form data, create interfaces that represent the form state
- For API responses, create interfaces that match the response structure
- Global types should be in [src/types](mdc:src/types), component-specific types should be with the component

## Styling
- Use Tailwind CSS for styling components
- Follow the configuration in [tailwind.config.js](mdc:tailwind.config.js)
- For complex components that can't be easily styled with Tailwind, create module CSS files
- Keep styling consistent across similar components
- Use responsive design principles
- Follow the color scheme and design tokens defined in the project

## CSS Organization
- Use utility-first approach with Tailwind CSS
- For complex styling, extract to custom components
- Use CSS modules or styled components for component-specific styles
- Keep global styles minimal and focused on application-wide concerns
- Follow a mobile-first approach to responsive design

## Naming Conventions
- Use PascalCase for React component names
- Use camelCase for variables, functions, and methods
- Use PascalCase for TypeScript interfaces and types
- Use UPPER_CASE for constants
- Use kebab-case for CSS class names and file names
- File naming should match the component/function name

## Component Structure
- Order imports: React, third-party libraries, internal modules
- Order component sections: types, constants, component function, helper functions
- Define prop interfaces before the component
- Export component as default at the end of the file
- For complex components, use the pattern:
  ```typescript
  // Imports
  // Types
  // Constants/helpers
  // Component
  // Export
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NehoraiHadad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
