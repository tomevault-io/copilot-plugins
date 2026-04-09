## otter

> All formatting rules (charset, indentation, line endings, etc.) are defined in `.editorconfig` at the repository root. Refer to that file for the configuration.


# Otter Framework - TypeScript/JavaScript Code Style

## General Formatting

All formatting rules (charset, indentation, line endings, etc.) are defined in `.editorconfig` at the repository root. Refer to that file for the configuration.

## TypeScript / JavaScript Conventions

- **Always write JSDoc comments** for methods and properties using `/** ... */` pattern
- **Properties must have the most restricted type possible**:
  ```typescript
  // Bad
  private type: string;

  // Good
  private type: 'A' | 'B';
  ```
- **Imports must be at the top of the file** - never in the middle of code
- **Follow existing ESLint rules** defined in `packages/@o3r/eslint-config`
- **Use `const` by default**, `let` only when reassignment is needed, never `var`

## File Organization

- **One class/interface per file** when possible
- **Keep files focused** - split large files into smaller, focused modules
- **Co-locate tests** with source files (e.g., `foo.service.ts` and `foo.service.spec.ts`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AmadeusITGroup)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AmadeusITGroup)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
