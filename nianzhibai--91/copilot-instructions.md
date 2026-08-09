## 91

> - Do not modify the README.md file in the project root directory unless explicitly requested.

# Development Guidelines

## Documentation Rules

- Do not modify the README.md file in the project root directory unless explicitly requested.

## Bug Fixing Principles

- When a bug is discovered, do not immediately apply a temporary patch.
- First analyze the root cause of the issue.
- Consider the problem from a system-level perspective:
  - Is the current architecture causing this issue?
  - Is there a missing abstraction or incorrect responsibility separation?
  - Does another part of the system need optimization or redesign?
- Prefer fundamental fixes that improve system reliability and maintainability over short-term workarounds.
- Avoid introducing special cases or hacks unless there is a clear justification.

## Before Making Changes

Before modifying code:

1. Understand the existing architecture.
2. Identify the affected components.
3. Explain the root cause of the problem.
4. Consider whether the issue indicates a broader design problem.
5. Only then implement the fix.

---
> Source: [nianzhibai/91](https://github.com/nianzhibai/91) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
