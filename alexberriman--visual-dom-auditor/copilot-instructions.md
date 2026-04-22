## visual-dom-auditor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@CLAUDE-CUSTOM.md
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Node fullstack project with automated testing and quality checks.

## Code Style Conventions

- **Case Style**: kebab-case
- **Indentation**: spaces (2 spaces)
- **Quotes**: double quotes
- **Line Length**: Maximum 100 characters
- **Trailing Commas**: Use trailing commas
- **Semicolons**: Use semicolons

### Component Structure

Components should be organized in directories with an index.ts file.
Example:
```
components/
  button/
    index.ts       # Exports the component
    button.tsx     # Component implementation
    button.test.tsx    # Tests (adjacent to implementation)
```

### Test Files

Test files should follow the pattern: `{name}.test.ts`

## Preferred Technologies

Use the following technologies in this project:

### Frontend

- **Framework**: React v19
- **Styling**: Tailwind v4
- **Data Fetching**: React-query
- **Documentation**: Storybook

### Code Organization & Architecture

- **Component Design**: Maximize reusability
  - Create small, composable, reusable components
  - Extract common patterns into shared components
  - Build a component hierarchy that promotes reuse
- **Component Documentation**: Every presentational component must have a Storybook story
  - Stories should cover all component states and variants
  - Include proper documentation of props and usage
- **Hooks Organization**: One hook per file
  - Each custom hook should have its own dedicated file
  - Organize hooks by feature/entity
  - Create separate hooks for each API operation (e.g., useCreateUser, useDeleteUser)
- **Data Fetching Pattern**: 
  - Create custom React Query hooks for each API endpoint
  - Organize queries by entity/resource
  - Leverage QueryClient for caching and background updates

### UI Design Philosophy (2025)

All UI components and interfaces should be CUTTING-EDGE, MODERN, and SEXY:

- **Visual Design**: Create interfaces that look like they're from 2025
  - Ultra-clean layouts with purposeful whitespace
  - Neumorphic or glassmorphic elements that add depth and dimension
  - Subtle shadows, gradients, and light effects
  - Floating elements and layered UI components

- **Interactions**: Design intuitive, fluid experiences
  - Micro-animations for state changes and transitions
  - Minimal friction in user workflows
  - Context-aware interfaces that anticipate user needs

- **Styling Approach**: Use Tailwind to create UNIQUE designs
  - Push creative boundaries with modern aesthetics
  - Utilize advanced color theory and typography
  - Aim for visually IMPRESSIVE and DISTINCTIVE interfaces


### Backend

- **Framework**: Express v5

### Utilities

- **Logging**: Pino
  - Configuration:
    - Log level: info
    - Pretty print: Enabled
    - Transport: pino-pretty
  - Do not use console.log - use appropriate log levels

### Testing

- **Unit Testing**: Vitest
  - Leveraging Vite for fast test execution
  - Do NOT use Jest configuration or dependencies
- **Component Testing**: Testing-library
- **Test Location**: Tests should be placed adjacent to implementation files
  - Do NOT use __tests__ directories

### Build Tools

- **Bundler**: Tsup
- **CI/CD**: github-actions

**All presentational ("dumb") components should have a corresponding Storybook story file.**



## Project Architecture

Follow a clear separation of concerns between frontend and backend. Use well-defined interfaces to communicate between layers.


## CLI Architecture

This CLI tool follows these architectural principles:

- **Command-line Interface**: Uses Commander.js for argument parsing
- **Browser Automation**: Playwright for DOM inspection and analysis
- **TypeScript**: Strong typing throughout the codebase
- **Error Handling**: Functional error handling with ts-results
- **Modular Design**: Separate concerns into focused modules
- **Pure Functions**: Maximize testability with pure functions


## Module Structure

The project should be organized as follows:

```
src/
  index.ts          # CLI entry point (#!/usr/bin/env node)
  auditor.ts        # Core DOM auditing functionality
  analyzers/        # Different DOM analyzers
    overlap.ts      # Detect overlapping elements
    padding.ts      # Check button padding
    spacing.ts      # Check element spacing
  types/            # TypeScript type definitions
    index.ts
    options.ts
    errors.ts
    issues.ts       # Types for detected issues
  utils/            # Utility functions
    viewport.ts
    error.ts
    validator.ts
    dom-helpers.ts  # DOM traversal helpers
  config/           # Configuration handling
    defaults.ts
```


## Error Handling Guidelines

When implementing error handling:

- Use ts-results for all fallible operations
- Return Result<T, E> from functions that might fail
- Provide descriptive error messages
- Never silence errors or use try/catch except at boundaries
- Exit with appropriate exit codes (0 for success, 1 for errors)
- Use console.error for error output
- Use console.log for success output

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexberriman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
