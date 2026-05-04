## ruler-windsurf-instructions-01

> - Don't use TypeScript enums.

- Don't use TypeScript enums.
- Don't export imported variables.
- Don't add type annotations to variables, parameters, and class properties that are initialized with literal expressions.
- Don't use TypeScript namespaces.
- Don't use non-null assertions with the `!` postfix operator.
- Don't use parameter properties in class constructors.
- Don't use user-defined types.
- Use `as const` instead of literal types and type annotations.
- Use either `T[]` or `Array<T>` consistently.
- Initialize each enum member value explicitly.
- Use `export type` for types.
- Use `import type` for types.
- Make sure all enum members are literal values.
- Don't use TypeScript const enum.
- Don't declare empty interfaces.
- Don't let variables evolve into any type through reassignments.
- Don't use the any type.
- Don't misuse the non-null assertion operator (!) in TypeScript files.
- Don't use implicit any type on variable declarations.
- Don't merge interfaces and classes unsafely.
- Don't use overload signatures that aren't next to each other.
- Use the namespace keyword instead of the module keyword to declare TypeScript namespaces.

### Style and Consistency
- Don't use global `eval()`.
- Don't use callbacks in asynchronous tests and hooks.
- Don't use negation in `if` statements that have `else` clauses.
- Don't use nested ternary expressions.
- Don't reassign function parameters.
- This rule lets you specify global variable names you don't want to use in your application.
- Don't use specified modules when loaded by import or require.
- Don't use constants whose value is the upper-case version of their name.
- Use `String.slice()` instead of `String.substr()` and `String.substring()`.
- Don't use template literals if you don't need interpolation or special-character handling.
- Don't use `else` blocks when the `if` block breaks early.
- Don't use yoda expressions.
- Don't use Array constructors.
- Use `at()` instead of integer index access.
- Follow curly brace conventions.
- Use `else if` instead of nested `if` statements in `else` clauses.
- Use single `if` statements instead of nested `if` clauses.
- Use `new` for all builtins except `String`, `Number`, and `Boolean`.
- Use consistent accessibility modifiers on class properties and methods.
- Use `const` declarations for variables that are only assigned once.
- Put default function parameters and optional function parameters last.
- Include a `default` clause in switch statements.
- Use the `**` operator instead of `Math.pow`.
- Use `for-of` loops when you need the index to extract an item from the iterated array.
- Use `node:assert/strict` over `node:assert`.
- Use the `node:` protocol for Node.js builtin modules.
- Use Number properties instead of global ones.
- Use assignment operator shorthand where possible.
- Use function types instead of object types with call signatures.
- Use template literals over string concatenation.
- Use `new` when throwing an error.
- Don't throw non-Error values.
- Use `String.trimStart()` and `String.trimEnd()` over `String.trimLeft()` and `String.trimRight()`.
- Use standard constants instead of approximated literals.
- Don't assign values in expressions.
- Don't use async functions as Promise executors.
- Don't reassign exceptions in catch clauses.
- Don't reassign class members.
- Don't compare against -0.
- Don't use labeled statements that aren't loops.
- Don't use void type outside of generic or return types.
- Don't use console.
- Don't use control characters and escape sequences that match control characters in regular expression literals.
- Don't use debugger.
- Don't assign directly to document.cookie.
- Use `===` and `!==`.
- Don't use duplicate case labels.
- Don't use duplicate class members.
- Don't use duplicate conditions in if-else-if chains.
- Don't use two keys with the same name inside objects.
- Don't use duplicate function parameter names.
- Don't have duplicate hooks in describe blocks.
- Don't use empty block statements and static blocks.
- Don't let switch clauses fall through.
- Don't reassign function declarations.
- Don't allow assignments to native objects and read-only global variables.
- Use Number.isFinite instead of global isFinite.
- Use Number.isNaN instead of global isNaN.
- Don't assign to imported bindings.
- Don't use irregular whitespace characters.
- Don't use labels that share a name with a variable.
- Don't use characters made with multiple code points in character class syntax.
- Make sure to use new and constructor properly.
- Don't use shorthand assign when the variable appears on both sides.
- Don't use octal escape sequences in string literals.
- Don't use Object.prototype builtins directly.
- Don't redeclare variables, functions, classes, and types in the same scope.
- Don't have redundant "use strict".
- Don't compare things where both sides are exactly the same.
- Don't let identifiers shadow restricted names.
- Don't use sparse arrays (arrays with holes).
- Don't use template literal placeholder syntax in regular strings.
- Don't use the then property.
- Don't use unsafe negation.
- Don't use var.
- Don't use with statements in non-strict contexts.
- Make sure async functions actually use await.
- Make sure default clauses in switch statements come last.
- Make sure to pass a message value when creating a built-in error.
- Make sure get methods always return a value.
- Use a recommended display strategy with Google Fonts.
- Make sure for-in loops include an if statement.
- Use Array.isArray() instead of instanceof Array.
- Make sure to use the digits argument with Number#toFixed().
- Make sure to use the "use strict" directive in script files.

### Next.js Specific Rules
- Don't use `<img>` elements in Next.js projects.
- Don't use `<head>` elements in Next.js projects.
- Don't import next/document outside of pages/_document.jsx in Next.js projects.
- Don't use the next/head module in pages/_document.js on Next.js projects.

### Testing Best Practices
- Don't use export or module.exports in test files.
- Don't use focused tests.
- Make sure the assertion function, like expect, is placed inside an it() function call.
- Don't use disabled tests.

## Common Tasks
- `npx ultracite init` - Initialize Ultracite in your project
- `npx ultracite fix` - Format and fix code automatically
- `npx ultracite check` - Check for issues without fixing

## Example: Error Handling
```typescript
// ✅ Good: Comprehensive error handling
try {
  const result = await fetchData();
  return { success: true, data: result };
} catch (error) {
  console.error('API call failed:', error);
  return { success: false, error: error.message };
}

// ❌ Bad: Swallowing errors
try {
  return await fetchData();
} catch (e) {
  console.log(e);
}
```



<!-- Source: .ruler/bts.md -->

# Better-T-Stack Project Rules

This is a easyapp-fullstack-template project created with Better-T-Stack CLI.

## Project Structure

This is a monorepo with the following structure:


- **`apps/server/`** - Backend server (Hono)

- **`apps/native/`** - React Native mobile app (with NativeWind)

## Available Scripts

- `pnpm run dev` - Start all apps in development mode
- `pnpm run dev:server` - Start only the server
- `pnpm run dev:native` - Start only the native app

## Database Commands

All database operations should be run from the server workspace:

- `pnpm run db:push` - Push schema changes to database
- `pnpm run db:studio` - Open database studio
- `pnpm run db:generate` - Generate Drizzle files
- `pnpm run db:migrate` - Run database migrations

Database schema files are located in `apps/server/src/db/schema/`

## API Structure

- oRPC endpoints are in `apps/server/src/api/`
- Client-side API utils are in `apps/web/src/utils/api.ts`

## Authentication

Authentication is enabled in this project:
- Server auth logic is in `apps/server/src/lib/auth.ts`
- Native app auth client is in `apps/native/src/lib/auth-client.ts`

## Adding More Features

You can add additional addons or deployment options to your project using:

```bash
pnpx create-better-t-stack
add
```

Available addons you can add:
- **Documentation**: Starlight, Fumadocs
- **Linting**: Biome, Oxlint, Ultracite
- **Other**: Ruler, Turborepo, PWA, Tauri, Husky

You can also add web deployment configurations like Cloudflare Workers support.

## Project Configuration

This project includes a `bts.jsonc` configuration file that stores your Better-T-Stack settings:

- Contains your selected stack configuration (database, ORM, backend, frontend, etc.)
- Used by the CLI to understand your project structure
- Safe to delete if not needed
- Updated automatically when using the `add` command

## Key Points

- This is a Turborepo monorepo using pnpm workspaces
- Each app has its own `package.json` and dependencies
- Run commands from the root to execute across all workspaces
- Run workspace-specific commands with `pnpm run command-name`
- Turborepo handles build caching and parallel execution
- Use `pnpx
create-better-t-stack add` to add more features later

---
> Source: [sunshineLixun/ShipFullStack](https://github.com/sunshineLixun/ShipFullStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
