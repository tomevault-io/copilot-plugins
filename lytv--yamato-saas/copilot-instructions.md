## yamato-saas

> description: TypeScript Type Safety Standards

---
description: TypeScript Type Safety Standards
globs: 
alwaysApply: true
---
 ---
description: TypeScript Type Safety Standards
globs: null
alwaysApply: true
---
 # TypeScript Type Safety Standards

## Context
- When writing TypeScript code across the codebase
- When reviewing TypeScript code changes
- When maintaining type safety in the project

## Requirements

### 1. Strict Type Checking
- Enable all strict compiler options in `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "strict": true,
      "noImplicitAny": true,
      "strictNullChecks": true,
      "strictFunctionTypes": true,
      "strictBindCallApply": true,
      "strictPropertyInitialization": true,
      "noImplicitThis": true,
      "useUnknownInCatchVariables": true,
      "alwaysStrict": true,
      "noUnusedLocals": true,
      "noUnusedParameters": true,
      "exactOptionalPropertyTypes": true,
      "noImplicitReturns": true,
      "noFallthroughCasesInSwitch": true,
      "noUncheckedIndexedAccess": true,
      "noImplicitOverride": true,
      "allowUnusedLabels": false,
      "allowUnreachableCode": false
    }
  }
  ```

### 2. Type Definitions
- Every variable, parameter, and return type must have explicit type annotations
- Avoid using `any` type - use `unknown` for truly unknown types
- Use `void` for functions that don't return values
- Use `never` for functions that never return
- Use `Record<string, unknown>` instead of `object` for object types
- Use `unknown` instead of `any` for external data

### 3. Interface and Type Usage
- Prefer interfaces for object shapes that can be extended
- Use type aliases for unions, intersections, and mapped types
- Make interfaces readonly when possible
- Use discriminated unions for complex state management
- Avoid empty interfaces

### 4. Generic Types
- Use generics to create reusable components
- Constrain generic parameters when possible
- Use generic constraints to ensure type safety
- Avoid using `any` in generic type parameters

### 5. Type Assertions
- Avoid type assertions (`as`) when possible
- Use type guards instead of type assertions
- When necessary, use `as const` for literal types
- Document why type assertions are needed

### 6. Null and Undefined Handling
- Use optional chaining (`?.`) for safe property access
- Use nullish coalescing (`??`) for default values
- Use type guards to narrow types
- Handle null and undefined explicitly

### 7. Function Types
- Use function type declarations for callbacks
- Use arrow functions for type inference
- Define return types explicitly for complex functions
- Use `void` for event handlers that don't return values

### 8. Class Types
- Define property types in class declarations
- Use access modifiers (public, private, protected)
- Implement interfaces explicitly
- Use abstract classes for shared behavior

## Examples

### ✅ Good Practices
```typescript
// Explicit type annotations
const name: string = 'John';
const age: number = 30;

// Proper interface usage
interface User {
  readonly id: string;
  name: string;
  email: string;
  role: UserRole;
}

// Type guards
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value
  );
}

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Proper null handling
const user = data?.user ?? defaultUser;
```

### ❌ Bad Practices
```typescript
// Avoid any
const data: any = getData();

// Avoid type assertions
const user = data as User;

// Avoid empty interfaces
interface Empty {}

// Avoid implicit any
function process(data) {
  return data;
}
```

## Critical Rules
1. Never use `any` type without explicit justification
2. Always define return types for functions
3. Use type guards instead of type assertions
4. Handle null and undefined explicitly
5. Use strict compiler options
6. Document complex type definitions
7. Use generics with proper constraints
8. Maintain consistent type naming conventions

## Enforcement
- ESLint rules should be configured to enforce these standards
- TypeScript compiler should be set to strict mode
- Code reviews should verify type safety
- Automated tests should include type checking
- CI/CD pipeline should include type checking step

WHENEVER YOU USE THIS RULE, START YOUR MESSAGE WITH THIS RULE NAME.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lytv)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lytv)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
