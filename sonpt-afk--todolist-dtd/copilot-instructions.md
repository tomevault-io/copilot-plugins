## todolist-dtd

> - Domain-based folder structure, not technical grouping

# GEMINI.md - React Clean Code Principles

## Architecture
- Domain-based folder structure, not technical grouping
- Absolute imports with @ aliases  
- Abstract external libraries in custom wrappers
- Keep things close to where they're used

## Components
- Function components only, TypeScript strict mode
- Max 5 props, destructure at top level
- Business logic in custom hooks, not components
- Early returns for guard clauses and error states
- Component size under 50 lines
- Named components, never anonymous

## Functions & Clean Code
- **Single Responsibility**: One clear purpose per function
- **Descriptive Names**: Use action verbs (fetchUser, validateEmail, calculateTotal)
- **Early Returns**: Guard clauses first, happy path obvious
- **Limit Parameters**: Max 3-4 params, group related ones into objects
- **Pure When Possible**: No side effects, predictable outputs
- **No Boolean Flags**: Use enums or separate functions instead
- **Return Results**: Use Result Pattern for expected errors, exceptions for unexpected
- **No Magic Values**: Extract numbers/strings into named constants

## State Management  
- React Query/SWR for server state
- useReducer for complex related state
- Context API only for global config/theme
- Multiple focused contexts, not single global

## Performance
- Measure before optimizing
- useMemo/useCallback only when profiled bottlenecks found
- Code splitting for routes and heavy features
- Optimize business logic first

## Testing
- Integration tests > unit tests > E2E tests
- AAA pattern (Arrange-Act-Assert)
- Test user behavior, not implementation details
- Test descriptions as user stories

## Code Quality
- TypeScript strict mode, no `any` types
- Props explicitly typed with interfaces
- ESLint + Prettier enforced in CI
- Helper functions outside components if no closure needed

## File Organization
- Component file structure (top to bottom):
- 1. Imports
- 2. Types/Interfaces  
- 3. Constants (no magic numbers/strings)
- 4. Helper functions (outside component)
- 5. Custom hooks
- 6. Main component with early returns
- 7. Default export

## Error Handling
- Error boundaries for component trees
- Suspense for async operations  
- Result objects for expected failures
- Clear error messages with recovery actions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sonpt-afk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sonpt-afk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
