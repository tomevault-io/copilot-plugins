## ts-rules

> TypeScript & React code styling

## TypeScript & React Style Guide

### Types
- Use `type` over `interface` for consistency
- Avoid `any`, use `unknown` for ambiguous types
- Use template literal types for string patterns
- Prefer discriminated unions over optional properties
- Use `ReadonlyArray<T>` and `Readonly<T>` for immutability
- Generic type params should start with T: `TRequest`, `TResponse`

### Functions
- Single responsibility
- Pure functions preferred
- Use single object parameter for multiple args
- Explicitly type return values
- Avoid side effects

### Variables
- Use `as const` for constants
- Use discriminated unions over boolean flags
- Use `null` for explicit no value, `undefined` for non-existent
- Prefer named exports

### Naming
- PascalCase: Types, Components
- camelCase: variables, functions
- UPPER_CASE: constants
- Boolean vars: prefix with `is/has`
- Avoid abbreviations
- React callbacks: `on*` props, `handle*` handlers
- React hooks: prefix with `use`

### React Specifics
- Prefer required props over optional
- Avoid props to state (use `initial` prefix if needed)
- Container components: suffix with `Container` or `Page`
- UI components: pure presentation only
- Custom hooks must return objects
- Data fetching only in containers

### File Organization
- Group by feature
- Collocate related code
- Use relative imports (`./`) within features
- Use absolute imports (`@/`) across features

### Comments
- Favor clear code over comments
- Use TSDoc for APIs and reusable code
- Comments explain "why" not "what"

### Testing
- Test business logic, not implementation
- Follow AAA pattern (Arrange, Act, Assert)
- Test descriptions: "should ... when ..."
- Query elements by role/label/text, not test IDs
- Avoid snapshot tests

---
> Source: [wlgitdev/SchemaToForm](https://github.com/wlgitdev/SchemaToForm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
