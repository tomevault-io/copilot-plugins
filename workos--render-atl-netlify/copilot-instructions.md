## render-atl-netlify

> description: Code style and formatting guidelines

---
description: Code style and formatting guidelines
globs: ["**/*"]
alwaysApply: false
---

# Code Style & Formatting

## Formatting Tools
- Follow the existing ESLint configuration in `eslint.config.js`
- Use Prettier for code formatting (settings in `.editorconfig`)
- Run formatting before commits
- Use consistent indentation (tabs)

## Code Quality
- Write clear, concise comments for complex logic
- Explain business logic and non-obvious code
- Keep comments up to date with code changes
- Use meaningful variable and function names

## Error Handling
- Use try-catch blocks for async operations
- Provide meaningful error messages
- Handle edge cases appropriately
- Implement proper fallback behavior

## Performance Considerations
- Avoid unnecessary computations in render cycles
- Use appropriate data structures for the task
- Consider memory usage for large datasets
- Optimize bundle size by avoiding unnecessary imports

## Code Comments
- Document component props with JSDoc when helpful
- Explain complex algorithms or business logic
- Use TODO comments for future improvements
- Remove obsolete comments during refactoring

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/workos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
