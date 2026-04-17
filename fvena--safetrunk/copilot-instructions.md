## safetrunk

> Guidelines for applying functional programming principles in this TypeScript library.

# Functional Programming in TypeScript

Guidelines for applying functional programming principles in this TypeScript library.

## Core Principles

- Prefer pure functions without side effects
- Treat data as immutable
- Use function composition for complex operations
- Avoid shared mutable state
- Separate data and behavior
- Use recursion appropriately (with stack considerations)

## Immutability

- Use `const` by default and `readonly` for types that shouldn't be modified
- Avoid methods that mutate data (push, pop, splice)
- Implement copies for modifications (spread, Object.assign)
- Use Readonly<T> for function parameters that shouldn't be modified
- Implement deep freeze for complex immutable objects

## First-Class Functions

- Pass functions as parameters when appropriate
- Implement functions that return functions (closures)
- Use function factories for configuration
- Prefer arrow functions to preserve context
- Use correctly typed callbacks

## Functional Composition

- Compose small functions to create complex behaviors
- Implement pipe and compose functions for chaining operations
- Use currying for partially applied functions
- Use point-free style when it improves readability
- Prefer expressions over statements in functional operations

## Collection Handling

- Use map, filter, reduce instead of imperative loops
- Avoid unnecessary intermediate operations in functional chains
- Implement fold and scan for accumulations
- Use flatMap for flattening with transformation
- Preserve immutability in collection operations

## Functional Types

- Implement Option/Maybe types for optional values
- Use Either/Result types for fallible operations
- Implement monads for chained operations
- Use algebraic data types (ADTs) when appropriate
- Implement pattern matching with discriminated unions

## Flow Control & Asynchrony

- Use conditional expressions instead of statements
- Avoid exceptions - use Result/Either types for errors
- Use Promises and async/await with functional handling
- Create combinators for complex async operations
- Implement consistent error handling in async flows

## Optimization

- Implement memoization for expensive pure functions
- Use lazy evaluation for costly calculations
- Implement trampolines for safe recursion
- Use persistent data structures for efficient operations
- Balance functional purity with performance where necessary

## Interoperability

- Create functional adapters for imperative APIs
- Maintain clear boundaries between functional and non-functional code
- Implement interoperability with RxJS or other functional libraries
- Design public APIs that support functional usage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fvena) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
