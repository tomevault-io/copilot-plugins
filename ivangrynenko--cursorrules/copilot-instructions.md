## debugging-standards

> Standards for debugging and error handling

# Debugging Standards

Ensures proper debugging practices and error handling.

## Rule Details

- **Name:** debugging_standards

- **Description:** Enforce standards for debugging and error handling

## Filters
- file extension pattern: `\\.(php|js|ts|vue|jsx|tsx|py)$`

## Enforcement Checks
- Conditions:
  - pattern `console\\.log\\(|print_r\\(|var_dump\\(` – Replace debug statements with proper logging
  - pattern `catch\\s*\\([^)]*\\)\\s*\\{\\s*\\}` – Implement proper error handling in catch blocks

## Suggestions
- Guidance:
Debugging Best Practices:
- Address root causes, not symptoms
- Add descriptive logging messages
- Create isolated test functions
- Implement comprehensive error handling
- Use appropriate logging levels
- Add context to error messages
- Consider debugging tools integration

## Metadata
- Priority: high
- Version: 1.1

 

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
