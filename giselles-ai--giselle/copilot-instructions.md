## naming-guide

> Naming convention guide for Giselle codebase

# Naming Guide

## Case Conventions

1. **File Naming: kebab-case**
   - All filenames should use kebab-case (lowercase with hyphens)
   - Examples:
     - ✅ `user-profile.ts`
     - ✅ `api-client.tsx`
     - ❌ `UserProfile.ts`
     - ❌ `apiClient.tsx`

2. **Component Naming: PascalCase**
   - React components and classes should use PascalCase
   - Examples:
     - ✅ `UserProfile`
     - ✅ `ApiClient`
     - ❌ `userProfile`
     - ❌ `user-profile`

3. **Variable Naming: camelCase**
   - Variables, functions, and methods should use camelCase
   - Examples:
     - ✅ `userEmail`
     - ✅ `isActive`
     - ❌ `UserEmail`
     - ❌ `user-email`

## Core Principles

1. **Readability First**
   - Names should be self-explanatory and immediately understandable
   - Avoid abbreviations unless they are widely known
   - Use descriptive names that explain the purpose or behavior

2. **Function Naming**
   - Use verbs or verb phrases for function names
   - Names should clearly indicate what the function does
   - Avoid ambiguous names that could lead to misuse
   - Examples:
     - ✅ `calculateTotalPrice()`
     - ✅ `validateUserInput()`
     - ❌ `process()` (too vague)
     - ❌ `doStuff()` (unclear purpose)

3. **Variable Naming**
   - Use nouns or noun phrases for variable names
   - Names should describe what the variable represents
   - Avoid single-letter variables except in very short scopes
   - Examples:
     - ✅ `userEmail`
     - ✅ `isActive`
     - ❌ `x` (unless in a mathematical context)
     - ❌ `temp` (unless truly temporary)

4. **Boolean Variables and Functions**
   - Use prefixes like `is`, `has`, `can`, `should` for both variables and functions returning boolean values
   - Make the true/false meaning clear
   - For variables examples:
     - ✅ `isEnabled`
     - ✅ `hasPermission`
     - ❌ `status` (ambiguous boolean)
   - For function examples:
     - ✅ `isTriggerRequiringCallsign()`
     - ✅ `hasActiveSubscription()`
     - ❌ `requiresCallsign()` (less clear about boolean return type)
     - ❌ `checkActive()` (ambiguous about what's returned)

5. **Consistency**
   - Follow consistent naming patterns across the codebase
   - Use the same terms for the same concepts
   - Maintain consistent casing (camelCase, PascalCase, etc.)

## Best Practices

- When someone reads your code, they should understand the purpose without additional context
- Function names should make it clear how to use them correctly
- Avoid names that could be misinterpreted or used incorrectly
- Consider the perspective of other developers who will maintain or use your code

---
> Source: [giselles-ai/giselle](https://github.com/giselles-ai/giselle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
