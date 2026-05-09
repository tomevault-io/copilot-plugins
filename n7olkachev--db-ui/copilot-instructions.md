## verify-changes

> Always verify your changes by running linting and type checking before declaring them complete. This ensures:


# Verify Changes with Linting and Type Checking

## Description

Always verify your changes by running linting and type checking before declaring them complete. This ensures:

1. No TypeScript errors
2. No linting issues
3. No runtime errors from type mismatches

## Process

1. After making changes, run your linter tool or you don't have any:

   ```bash
   # Check TypeScript types
   npx tsc --noEmit

   # Run linter
   npm run lint
   ```

2. Fix any errors before proceeding
3. Never assume changes are complete without verification

## Examples

### Good

```bash
# Make changes to code
# Run verification
npx tsc --noEmit
npm run lint
# Fix any errors
# Then proceed with implementation
```

### Bad

```bash
# Make changes to code
# Assume everything works without verification
# Continue with implementation
```

## Rationale

TypeScript and linting errors can indicate:

1. Missing dependencies
2. Incorrect imports
3. Type mismatches
4. Potential runtime errors

Always verify changes to prevent these issues from reaching production.

DO NOT run npm run dev yourself, only run llint scripts.

---
> Source: [n7olkachev/db-ui](https://github.com/n7olkachev/db-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
