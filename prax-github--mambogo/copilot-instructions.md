## mambogo

> - Fix ALL linting errors immediately - never defer warnings

---
description:
# Code Quality Rules - Professional Software Engineering Standards

## Code Quality Gates (Mandatory)
- Fix ALL linting errors immediately - never defer warnings
- Remove unused imports, variables, methods, and fields immediately when detected
- Use read_lints tool after every 2-3 file modifications
- Ensure proper error handling (try-catch) in all service methods
- Maintain consistent naming: camelCase variables, PascalCase classes, UPPER_SNAKE_CASE constants

## Incremental Development Protocol
- Make changes to maximum 2-3 files at once before verification
- Compile and verify after each new class or significant method addition
- Never accumulate more than 5 unverified changes
- Use todo_write to track progress in real-time, not at the end

## Error Handling Discipline
- When compilation errors occur: STOP adding new features until ALL errors are resolved
- Read complete error messages - don't just glance and make assumptions
- Fix errors in dependency order: classes with no dependencies first
- Use systematic debugging: understand root cause before making changes
- Document what was learned from error resolution for future reference

## File Organization Standards
- Keep classes focused on single responsibility
- Avoid classes with more than 200 lines
- Break complex methods into smaller, focused methods
- Use clear, descriptive method and variable names

## Import Management
- Remove unused imports immediately
- Use specific imports rather than wildcard imports
- Organize imports by package (java.*, org.*, com.*, project specific)
- Don't import classes that are only used once in comments

## Variable and Method Naming
- Use camelCase for variables and methods
- Use PascalCase for classes
- Use UPPER_SNAKE_CASE for constants
- Avoid abbreviations unless they're widely understood

## Documentation Standards
- Add JavaDoc for all public methods and classes
- Include @param and @return documentation
- Document any complex algorithms or business logic
- Explain WHY, not just WHAT the code does

globs:
- "**/*.java"
- "**/*.js"
- "**/*.ts"
- "**/*.jsx"
- "**/*.tsx"
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prax-github) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
