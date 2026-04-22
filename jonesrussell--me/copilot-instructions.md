## me

> - Use TypeScript for all new code

# Development Standards

## Core Principles
- Use TypeScript for all new code
- Document code thoroughly with JSDoc comments
- Follow SOLID principles
- Write pure functions when possible
- Implement strict type checking
- Validate props at runtime
- Handle errors consistently

## Code Style
- Use semicolons consistently
- Prefer `const` declarations
- Limit lines to 100 characters
- Use 2-space indentation
- Follow naming conventions:
  - Interface names start with "I"
  - Type names end with "Type"
  - Use descriptive names for tests

## Architecture
- Organize code by feature
- Separate UI components from business logic
- Keep services stateless
- Use dependency injection
- Minimize component logic
- Maintain test independence

## Type System
- Store shared types in `src/types`
- Prefer interfaces over type aliases
- Document complex types
- Export types separately from implementations

## Services
- Use factory patterns for services
- Implement comprehensive error handling
- Add request caching where appropriate
- Include retry logic and timeouts
- Use middleware for cross-cutting concerns

## State Management
- Use runes for state management (`$state`, `$derived`, `$effect`)
- Stores use `.svelte.ts` extension (required for runes)
- Stores export state objects directly using `$state`
- Components access state directly (no `$` prefix needed)
- Keep state focused and lightweight
- Test state updates thoroughly
- Use `$props` for component properties
- Implement proper cleanup in effects
- Services are pure functions - they return data, they don't manage state

## Svelte Development
For Svelte-specific patterns and best practices, refer to:
- [Svelte 5 Standards](mdc:svelte5-standards.mdc) for runes and modern features
- [Component Styling Standards](mdc:component-styling.mdc) for styling patterns
- [Testing Standards](mdc:testing-standards.mdc) for testing practices

## Svelte Patterns
- Use runes for state management (`$state`, `$derived`, `$effect`)
- Use `$props` for component properties
- Follow accessibility guidelines
- Implement error boundaries
- Use proper TypeScript types

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonesrussell) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
