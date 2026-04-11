## haskell-literate

> generates `helloworld`, you would run:

# Literate Haskell Project

This project uses Literate Haskell for writing and documenting Haskell code.

## Build and Run

To build the executables defined in the `.lhs` files:

```bash
make all
```

You can then run the generated executables. For example, if `doc/helloworld.lhs`
generates `helloworld`, you would run:

```bash
./helloworld
```

## Generate Documentation

To generate HTML documentation from the literate Haskell files:

```bash
make html
```

To generate PDF documentation:

```bash
make pdf
```

The generated `html` and `pdf` files will be in the `doc/` directory (e.g.,
`doc/helloworld.html`, `doc/bird.pdf`).

## Cleaning

To remove intermediate build files:

```bash
make clean
```

To remove all generated files (executables, HTML, PDF, and intermediate files):

```bash
make cleanall
```

## Tooling Setup

This project relies on `hlint` for linting and `stylish-haskell` for formatting.
These tools help maintain code quality and consistency.

### Installation

You can install them using either `cabal` or `stack`:

**With Cabal:**
```bash
cabal update
cabal install hlint stylish-haskell
```

**With Stack:**
```bash
stack install hlint stylish-haskell
```

Make sure the installation directory is in your system's `PATH`.

## General Haskell Programming Review Agent

### Description

An agent specialised in reviewing Haskell codebases using modern functional
programming idioms. It ensures clean, composable code; strong type safety;
proper documentation; and adherence to best practices using tools such as
HLint, Haddock, and (when relevant) Cabal/Stack conventions.

The agent enforces:

- Pure functional programming
- Clear function boundaries
- Small, composable definitions
- Proper type signatures
- Correct, structured documentation
- Clean-code principles appropriate for functional languages

### Core Principles

Use "General Programming Review Agent" as a base.

### Coding Style

* Use 2 spaces for indentation.
* Limit lines to 80 characters.
* Use camelCase for variable and function names.
* Use PascalCase for type and data constructors.
* Use single characters for generic type variables (e.g., `a`, `b`, `m`).
* Use [hlint](https://github.com/ndmitchell/hlint) to enforce style guidelines and suggest improvements.
* Use [stylish-haskell](https://github.com/haskell/stylish-haskell) for consistent formatting.

### Functional Programming (FP) Foundations

- Encourage pure, total functions.
- Avoid partial functions (`head`, `tail`, `fromJust`, etc.) unless justified.
- Promote:
  - higher-order functions (`map`, `foldr`, `foldl'`, `traverse`, `sequence`)
  - applicative/monadic patterns where appropriate
  - function composition (`(.)`) and pipelines
  - point-free style when it improves clarity
- Prefer algebraic data types and pattern matching.
- Encourage immutability and referential transparency.

### Type Safety & Type System Usage

- Require explicit type signatures for top-level functions.
- Encourage:
  - newtypes over type synonyms when providing semantic meaning
  - sum types for branching logic rather than booleans or magic values
  - record syntax for clarity
  - deriving strategies (`deriving stock`, `deriving newtype`)
- Promote strong type design and small domain-specific types.

### Clean Code & Architecture

- Functions should be small, focused, and composable.
- Modules should adhere to Single Responsibility Principle.
- Avoid:
  - deeply nested pattern matches
  - overly large case expressions
  - unnecessary monad stacks
  - mixing pure and effectful logic in the same function
- Prefer pure logic at the core and constrained effects at the boundaries.
- Suggest decomposition of complex monadic pipelines.

### Documentation Standards (Haddock)

Every top-level function and type should include Haddock-style documentation:

- Summary description
- Parameter explanation via `-- ^`
- Return value description
- Example usage via `>>>`
- When applicable, document invariants and laws

Examples:

```haskell
-- | Compute the average of a list of numbers.
average :: Fractional a => [a] -> a
average xs = sum xs / fromIntegral (length xs)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frankhjung)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/frankhjung)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
