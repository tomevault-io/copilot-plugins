## ai-template

> Project structure, directory organization, and file size guidelines


# Project Structure Guidelines

## Directory Organization

### Root Level Structure

```
project/
├── apps/                 # Application packages (monorepo)
├── shared/              # Shared libraries and utilities
├── config/              # Configuration files
├── docs/                # Project documentation
├── scripts/             # Development and build scripts
├── tests/               # All test files
│   ├── e2e/            # End-to-end tests
│   ├── unit/           # Unit tests
│   └── integration/    # Integration tests
├── test-images/         # Playwright test screenshots
├── demo-videos/         # Demo recordings
└── tickets/             # Project tickets and planning
```

### Frontend Application Structure

```
src/
├── components/          # Reusable UI components
│   ├── common/         # Shared components
│   ├── forms/          # Form components
│   └── layout/         # Layout components
├── pages/              # Next.js pages or route components
├── hooks/              # Custom React hooks
├── services/           # API and external service integrations
├── utils/              # Utility functions and helpers
├── types/              # TypeScript type definitions
├── styles/             # Global styles and themes
├── contexts/           # React context providers
├── constants/          # Application constants
└── lib/                # Third-party library configurations
```

### Backend Application Structure (NestJS)

```
src/
├── modules/            # Feature modules
│   ├── users/         # User module
│   ├── auth/          # Authentication module
│   └── products/      # Product module
├── common/            # Shared code
│   ├── decorators/    # Custom decorators
│   ├── filters/       # Exception filters
│   ├── guards/        # Authentication guards
│   ├── interceptors/  # HTTP interceptors
│   ├── pipes/         # Validation pipes
│   └── dto/           # Common DTOs
├── config/            # Configuration modules
├── database/          # Database schemas and migrations
├── utils/             # Utility functions
└── main.ts           # Application entry point
```

### Module Structure (Feature-based)

```
feature-module/
├── dto/               # Data Transfer Objects
├── entities/          # Database entities/schemas
├── controllers/       # HTTP controllers
├── services/          # Business logic services
├── repositories/      # Data access layer (if needed)
├── guards/            # Module-specific guards
├── tests/             # Module-specific tests
└── module.ts          # Module definition
```

## File Organization Guidelines

### Naming Conventions

#### Files and Directories

- **Directories**: `kebab-case` (e.g., `user-profile/`)
- **React Components**: `PascalCase.tsx` (e.g., `UserProfile.tsx`)
- **Hooks**: `camelCase.ts` starting with 'use' (e.g., `useAuth.ts`)
- **Services**: `camelCase.service.ts` (e.g., `userAuth.service.ts`)
- **Utilities**: `camelCase.ts` (e.g., `dateHelper.ts`)
- **Types**: `camelCase.types.ts` (e.g., `user.types.ts`)
- **Constants**: `UPPER_SNAKE_CASE.ts` (e.g., `API_ENDPOINTS.ts`)

#### Test Files

- **Unit Tests**: `*.test.ts` or `*.spec.ts`
- **E2E Tests**: `*.e2e.ts` or `*.e2e-spec.ts`
- **Component Tests**: `ComponentName.test.tsx`

### File Size Limits

#### Code Files

- **Soft Limit**: 300 lines of code
- **Hard Limit**: 500 lines of code
- **Action Required**: If a file exceeds 300 lines, refactor into smaller modules

#### Functions and Methods

- **Soft Limit**: 50 lines per function
- **Hard Limit**: 80 lines per function
- **Action Required**: Extract helper functions or break into smaller methods

#### Line Length

- **Preferred**: 100 characters per line
- **Hard Limit**: 120 characters per line
- **Solution**: Use proper line breaks and indentation

### File Content Organization

#### Import Order (TypeScript/JavaScript)

```typescript
// 1. External library imports
import React from "react";
import { Button } from "@mui/material";

// 2. Internal module imports
import { UserService } from "../services";
import { validateInput } from "../utils";

// 3. Type imports (separate from value imports)
import type { User, UserRole } from "../types";

// 4. Relative imports
import "./ComponentName.styles.css";
```

#### Component File Structure

```typescript
// 1. Imports
import React, { useState, useEffect } from "react";
import type { ComponentProps } from "./ComponentName.types";

// 2. Types and interfaces
interface ComponentState {
  // ...
}

// 3. Constants
const DEFAULT_CONFIG = {
  // ...
};

// 4. Component definition
export const ComponentName: React.FC<ComponentProps> = ({ prop1, prop2 }) => {
  // 5. State and hooks
  const [state, setState] = useState<ComponentState>({});

  // 6. Event handlers
  const handleClick = () => {
    // ...
  };

  // 7. Effects
  useEffect(() => {
    // ...
  }, []);

  // 8. Render
  return <div>{/* JSX content */}</div>;
};

// 9. Default export (if needed)
export default ComponentName;
```

## Code Organization Principles

### Single Responsibility Principle

- Each file should have a single, well-defined purpose
- Components should handle only UI rendering and user interaction
- Services should handle business logic and API calls
- Utilities should provide pure functions

### DRY (Don't Repeat Yourself)

- Extract common logic into utility functions
- Create reusable components for repeated UI patterns
- Use constants for repeated values
- Share types across modules

### KISS (Keep It Simple, Stupid)

- Prefer simple, readable solutions over clever code
- Use descriptive variable and function names
- Avoid deep nesting (max 4 levels)
- Break complex operations into smaller steps

### Separation of Concerns

- Keep business logic separate from UI components
- Isolate API calls in service layers
- Separate data validation from business logic
- Keep styling separate from component logic

## Refactoring Guidelines

### When to Refactor

- File exceeds size limits
- Function becomes too complex
- Code is duplicated across files
- Business logic mixed with UI logic

### How to Refactor

1. **Extract Components**: Break large components into smaller ones
2. **Extract Hooks**: Move stateful logic to custom hooks
3. **Extract Services**: Move API calls and business logic to services
4. **Extract Utils**: Move pure functions to utility files
5. **Extract Constants**: Move magic numbers and strings to constants

### Large File Exceptions

Some files may legitimately be large:

- **Configuration files**: Environment configs, webpack configs
- **Schema files**: Database schemas, API schemas
- **Mock data files**: Test fixtures and mock data
- **Generated files**: Auto-generated code

For these files:

- Add a comment explaining why the file is large
- Organize content with clear sections and comments
- Consider splitting if possible without losing clarity

## Documentation Requirements

### README Files

- **Root README**: Project overview, setup instructions, architecture
- **Module READMEs**: Feature-specific documentation
- **Component READMEs**: Complex component usage and examples

### Code Comments

- **File Headers**: Brief description of file purpose
- **Complex Logic**: Explain why, not what
- **API Endpoints**: Document parameters and responses
- **Business Rules**: Explain domain-specific logic

### Type Documentation

- Use JSDoc comments for complex types
- Document generic type parameters
- Explain union type usage
- Document API response shapes

## Best Practices

### File Management

- Delete unused files regularly
- Keep dependencies up to date
- Use absolute imports for cleaner paths
- Organize imports consistently

### Asset Organization

- **Images**: Group by feature or common/shared
- **Styles**: Co-locate with components when possible
- **Fonts**: Store in public/assets/fonts
- **Icons**: Use icon libraries or SVG components

### Environment Management

- Separate configs for different environments
- Use environment variables for sensitive data
- Document all required environment variables
- Provide example .env files

This structure promotes maintainability, scalability, and team collaboration while keeping the codebase organized and efficient.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasonstcyrx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jasonstcyrx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
