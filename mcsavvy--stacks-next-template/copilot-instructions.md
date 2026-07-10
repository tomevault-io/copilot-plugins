## code-quality

> This project enforces high code quality through automated tooling and consistent patterns.


# Code Quality Standards

This project enforces high code quality through automated tooling and consistent patterns.

## Tooling

### Linting & Formatting
- **Biome**: Primary linter and formatter (configured in [biome.json](mdc:biome.json))
- **TypeScript**: Strict type checking enabled
- **Prettier**: Integrated with Biome for consistent formatting

### Scripts
```json
{
  "lint": "biome check",
  "format": "biome format --write"
}
```

## Code Standards

### File Organization
- **Consistent imports** - Group imports by type (external, internal, types)
- **Export patterns** - Use named exports for components, default for pages
- **File naming** - Use kebab-case for files, PascalCase for components

### TypeScript Standards
- **Strict mode** - No implicit any, strict null checks
- **Type definitions** - Define interfaces for all data structures
- **Generic usage** - Use generics for reusable components
- **Error handling** - Proper error types and handling

### React Patterns
- **Functional components** - Use function declarations over arrow functions
- **Hooks** - Custom hooks for reusable logic
- **Props** - Use proper TypeScript interfaces for props
- **State management** - Context for global state, local state for component state

## Code Style

### Naming Conventions
- **Variables**: camelCase (`userName`, `isConnected`)
- **Functions**: camelCase (`connectWallet`, `handleClick`)
- **Components**: PascalCase (`UserProfile`, `WalletButton`)
- **Files**: kebab-case (`user-profile.tsx`, `wallet-button.tsx`)
- **Constants**: UPPER_SNAKE_CASE (`API_URL`, `STORAGE_KEY`)

### Import Organization
```typescript
// 1. External libraries
import React from "react";
import { useWallet } from "@stacks/connect";

// 2. Internal imports
import { Button } from "@/components/ui/button";
import { useAuthSession } from "@/providers/auth-session-provider";

// 3. Type imports
import type { AppConfig } from "@/lib/config/client";
```

### Component Structure
```typescript
// 1. Imports
// 2. Types/Interfaces
// 3. Component
// 4. Exports
```

## Best Practices

1. **Single Responsibility** - Each function/component should have one purpose
2. **DRY Principle** - Don't repeat yourself, extract common logic
3. **Error Boundaries** - Implement proper error handling
4. **Performance** - Use React.memo, useMemo, useCallback appropriately
5. **Accessibility** - Include proper ARIA attributes and keyboard navigation
6. **Testing** - Write tests for critical functionality
7. **Documentation** - Include JSDoc for complex functions
8. **Security** - Validate all inputs, sanitize data

---
> Source: [Mcsavvy/stacks-next-template](https://github.com/Mcsavvy/stacks-next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
