## kjv-web

> - Use fp-ts for all data transformations and control flow

# KJV Web - Functional Programming Rules with fp-ts

## Core Principles

- Use fp-ts for all data transformations and control flow
- Prefer declarative over imperative code
- Use immutable data structures
- Handle errors and null values with monads (Option, Either, TaskEither)
- Compose functions using pipe and flow
- Avoid mutations and side effects

## Required fp-ts Patterns

### Data Transformation

- Use `pipe()` for data flow instead of method chaining
- Use `Array` module (`A.*`) instead of native array methods
- Use `Record` module (`R.*`) for object operations
- Use `String` module (`S.*`) for string operations

### Control Flow

- Use `Option` for nullable values instead of `null`/`undefined` checks
- Use `Either` for error handling instead of try/catch
- Use `TaskEither` for async operations that can fail
- Use `fold`, `match`, or `getOrElse` for pattern matching

### Function Composition

- Use `flow()` for function composition
- Use `pipe()` for data transformation pipelines
- Prefer curried functions for partial application
- Use `contramap` and `map` for transformations

## Code Examples

### ✅ Good (fp-ts style)

```typescript
import { pipe } from "fp-ts/function";
import * as A from "fp-ts/Array";
import * as O from "fp-ts/Option";
import * as E from "fp-ts/Either";

// Data transformation
const processUsers = (users: User[]) =>
  pipe(
    users,
    A.filter((user) => user.active),
    A.map((user) => ({ ...user, name: user.name.toUpperCase() })),
    A.sort(byName)
  );

// Null handling
const findUser = (id: string): O.Option<User> =>
  pipe(
    users,
    A.findFirst((user) => user.id === id)
  );

// Error handling
const validateUser = (data: unknown): E.Either<string, User> =>
  pipe(
    data,
    E.fromPredicate(isUser, () => "Invalid user data")
  );
```

### ❌ Avoid (imperative style)

```typescript
// Don't use native array methods
const activeUsers = users.filter((u) => u.active).map((u) => u.name);

// Don't use null checks
if (user !== null && user !== undefined) {
  return user.name;
}

// Don't use try/catch for validation
try {
  const user = parseUser(data);
  return user;
} catch (error) {
  return null;
}
```

## Required Imports

Always import fp-ts modules with consistent aliases:

```typescript
import { pipe, flow } from "fp-ts/function";
import * as A from "fp-ts/Array";
import * as O from "fp-ts/Option";
import * as E from "fp-ts/Either";
import * as TE from "fp-ts/TaskEither";
import * as R from "fp-ts/Record";
import * as S from "fp-ts/string";
import * as N from "fp-ts/number";
```

## React Components

- Use fp-ts patterns in component logic
- Handle loading states with `Option` or `Either`
- Use `pipe` for data transformations in render functions
- Prefer functional components with hooks

## Error Handling

- Never use `throw` - use `Either` or `TaskEither`
- Handle all possible failure cases explicitly
- Use `fold` or `match` to handle success/failure branches
- Provide meaningful error types, not just strings

## Async Operations

- Use `TaskEither` for async operations that can fail
- Use `Task` for async operations that always succeed
- Chain async operations with `chain` and `map`
- Handle promises with `TE.tryCatch`

## Type Safety

- Define custom types for domain objects
- Use branded types for IDs and validated data
- Prefer union types over enums
- Use `const assertions` for literal types

## Documentation

- Document all public functions with JSDoc
- Include examples for complex fp-ts transformations
- Explain the reasoning behind monadic choices
- Document error cases and their meanings

## File Organization

- Keep pure functions separate from React components
- Group related transformations in utility modules
- Export composed functions, not individual steps
- Use barrel exports for clean imports

## Performance

- Use `lazy` evaluation where appropriate
- Prefer `chain` over nested `map` calls
- Use `sequenceT` and `sequenceS` for parallel operations
- Consider `IO` for expensive computations

Remember: Every operation should be a pure function that can be composed with others. If you find yourself writing imperative code, step back and think about how to express it functionally with fp-ts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robcross1977) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
