## solid-core-template

> **Activation Mode:** Always On


# SolidJS Project Standards

**Activation Mode:** Always On

**Priority:** High

## Documentation First Rule

### Research Protocol

- **Documentation:** Before making changes, check `./.rag-docs/` for official documentation of each library/package or existing markdown conversation logs
- If no documentation exists, read library/package versions from `./package.json`
- Study documentation specific for that version from web, then implement solution based on those docs

## Package Management

- **Core Tool:** `yarn` (Mandatory)
- **Forbidden:** Never use npm or pnpm
- **Installation:** `yarn add <package>` (or `yarn add -D <package>` for dev dependencies)
- **Scripts:** Use `yarn build`, `yarn dev`, etc.

## Import Aliases & Paths

**Use defined import aliases in `./tsconfig.json`. If directories are not covered by aliases, create an MD file pointing out those directories.**

## Styling Requirements

- **Framework:** Must and only use Tailwind CSS for all styling
- **Forbidden:** No other CSS frameworks or inline styles

## SVG Usage

- Convert SVG files into components/modules instead of using them as media in image tags
- Use SVG as plain HTML/SVG code

## SolidJS Specific Patterns

### Reactive State Management

- Use `createSignal()` for reactive state
- Access signal values with function call: `const value = signal()`
- Update signals with setter: `setSignal(newValue)`

### Router Integration

- TanStack Router integrates with SolidJS reactivity system
- Route params, query params, and search params are all Accessor objects
- Always use function call syntax to access reactive router values

## Code Quality

### TypeScript Standards

- Use TypeScript for type safety
- Use `type` instead of `interface` for type definitions
- Follow existing code patterns and conventions
- Ensure all imports are at the top of files
- Maintain consistent code formatting
- **SolidJS Reactivity:** Access reactive values using function call syntax `value()` not direct property access
- **Accessor Pattern:** SolidJS uses Accessor objects for reactive state - always call them as functions

### Implementation Guidelines

- Follow existing code patterns and conventions
- Ensure all imports are at the top of files
- Use TypeScript for type safety
- Maintain consistent code formatting
- Use `type` instead of `interface` for type definitions

### TanStack Router with SolidJS

- **Route Parameters:** Always access route parameters using accessor pattern:

  ```typescript
  // Correct
  const params = Route.useParams();
  const postId = params().postId;

  // Incorrect - will cause TypeScript errors
  const { postId } = Route.useParams();
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/malinjr07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
