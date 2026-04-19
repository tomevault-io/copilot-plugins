## hackathon-frontend-main

> This rule defines the standard TypeScript naming conventions to be used across the codebase for consistency and improved code readability.

# TypeScript Naming Conventions

## Overview
This rule defines the standard TypeScript naming conventions to be used across the codebase for consistency and improved code readability.

## Glob
```
**/*.ts
**/*.tsx
```

## Rules

### Prefixes by Type

Use meaningful prefixes to indicate the nature of the declaration:

| Prefix | Type | Description | Example |
|--------|------|-------------|---------|
| `i` | Instance | Prefix for instance variables | `const iAuthUseCase = new AuthUseCase()` |
| `T` | Type | Prefix for type aliases | `type TUserPayload = { name: string; age: number }` |
| `I` | Interface | Prefix for interfaces | `interface IUser { id: number; email: string }` |
| `C` | Constant | Prefix for constants | `const CMAXLIMIT = 100` |
| `E` | Enum | Prefix for enumerations | `enum EUserRole { Admin = 'ADMIN' }` |

### Casing Conventions

1. **PascalCase**
   - Classes
   - Interfaces (with `I` prefix)
   - Type aliases (with `T` prefix)
   - Enums (with `E` prefix)

2. **camelCase**
   - Variables
   - Functions
   - Methods
   - Properties
   - Instances (with `i` prefix)

3. **UPPER_SNAKE_CASE**
   - Constants (with `C` prefix)

### Examples

#### Type Declarations
```typescript
// Interface (IPrefix + PascalCase)
interface IUser {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
}

// Type alias (TPrefix + PascalCase)
type TUserResponse = {
  user: IUser;
  token: string;
};

// Enum (EPrefix + PascalCase)
enum EUserRole {
  Admin = 'ADMIN',
  User = 'USER',
  Guest = 'GUEST',
}
```

#### Variables and Constants
```typescript
// Instance (iPrefix + camelCase)
const iAuthUseCase = new AuthUseCase();

// Regular variable (camelCase)
const userName = 'John Doe';

// Constant (CPrefix + UPPER_SNAKE_CASE)
const CMAX_RETRY_ATTEMPTS = 3;
const CAPI_URL = 'https://api.example.com';
```

#### Functions and Methods
```typescript
// Function (camelCase)
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

// Method in class (camelCase)
class OrderService {
  processOrder(order: IOrder): void {
    // Implementation
  }
}
```

### Generic Type Parameters

For generic type parameters, use single uppercase letters or descriptive PascalCase:

```typescript
// Single letter for simple generics
function identity<T>(arg: T): T {
  return arg;
}

// Descriptive PascalCase for more complex generics
interface IRepository<TEntity, TKey> {
  findById(id: TKey): Promise<TEntity | null>;
  save(entity: TEntity): Promise<TEntity>;
}
```

## Benefits

1. **Improved Readability**: Prefixes provide immediate recognition of a declaration's purpose
2. **IDE Support**: Enhanced autocomplete with distinctive prefixes
3. **Type Safety**: Easier identification of interfaces vs. types vs. classes
4. **Consistency**: Standardized naming across the codebase
5. **Maintainability**: Reduced cognitive load when reviewing and understanding code

## Exceptions

Exceptions to these rules must be documented and approved by the team lead or architecture owner. The documentation should include the reason for the exception and how it relates to the project's architecture.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Akshat0904) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
