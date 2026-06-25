## better-sol

> When encountering errors or unfamiliar APIs:

## Critical Rules

When encountering errors or unfamiliar APIs:

1. **For installed libraries in node_modules:**
   - Check the library source code directly in `node_modules/<package-name>`
   - Read the README, types, or source files to understand usage
   - Do NOT use MCP servers for libraries that are already installed

2. **For external libraries or new dependencies:**
   - Only search the internet when you genuinely need external information
   - Use MCP tools for official documentation when unavailable locally
   - Never guess syntax, parameters, or behavior

**Do NOT search the internet for:**

- Basic programming concepts
- Common language features
- Information available in the codebase
- Things you can determine by reading the code or types

## Tool Requirements

For **internal development only**, use **bun** and **bunx** exclusively.
Do **NOT** use npm, npx, pnpm, or yarn for internal development.

## Plan Mode

When operating in Plan Mode:

- Produce concise, execution-focused plans.
- Clarity is required; verbosity is not.
- Do not include explanations, prose, or filler.
- Output only concrete steps.
- Every plan **MUST** end with a section titled **"Unresolved Questions"**.
- Include only questions that materially block execution.
- If there are no blockers, explicitly write:

  **Unresolved Questions: None.**

Failure to follow this structure is not allowed.

## Repository Analysis

When analyzing an external repository:

- Clone the target repository into a **temporary directory** (e.g., `/tmp/repo-analysis-<name>`).
- Perform all analysis, searches, and reads from the cloned copy.
- Once analysis is complete, **delete the cloned directory** to free up disk space.
- Never clone repositories into the project working directory.

## Code Style & Generation Rules

All generated code must follow these rules:

- Code must be self-explanatory.
- Do not use comments.
- Intent must be expressed through naming, structure, and composition only.

Strict consistency is required across:

- File structure
- Module boundaries
- Naming conventions
- Architectural patterns

Before writing any code:

- Explore the existing project.
- Understand the directory layout, conventions, abstractions, and dependency patterns.

## Type Safety Requirements

All code must be **100% type safe** without exceptions:

- **NEVER use `any`** - Use `unknown` with proper type guards instead
- **NEVER use unsafe type casting** - Use type guards, discriminated unions, or generics
- **NEVER use `@ts-ignore` or `@ts-expect-error`** unless absolutely necessary and documented
- **ALWAYS use explicit types** for function parameters and return types
- **ALWAYS prefer strict null checks**
- **ALWAYS use discriminated unions** for union types with different shapes
- **ALWAYS validate external data** (API responses, user input) with runtime type checking (zod, valibot, etc.)
- **ALWAYS ensure end-to-end type safety** from API to UI components

## Software Engineering Principles

Apply these principles in all code:

### SOLID

- **S**ingle Responsibility: Each module/class/function does one thing well
- **O**pen/Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes must be substitutable for their base types
- **I**nterface Segregation: Prefer small, focused interfaces
- **D**ependency Inversion: Depend on abstractions, not concrete implementations

### KISS (Keep It Simple, Stupid)

- Write simple, readable code
- Avoid unnecessary complexity
- Solve the problem directly, not abstractly

### YAGNI (You Aren't Gonna Need It)

- Don't add functionality until it's actually needed
- Avoid premature abstraction
- Refactor when necessity arises

### DRY (Don't Repeat Yourself)

- Extract duplicated logic into reusable functions/modules
- Use shared utilities for common operations
- Avoid code duplication across packages

---
> Source: [better-sol/better-sol](https://github.com/better-sol/better-sol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
