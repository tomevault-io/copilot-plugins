## react-patterns

> React component patterns and hooks usage guidelines

# React Patterns and Best Practices

Ensures React components follow recommended patterns and hook usage.

<rule>
name: react_patterns
description: Enforce React component patterns and hooks best practices
filters:
  - type: file_extension
    pattern: "\\.(jsx|tsx)$"

actions:
  - type: enforce
    conditions:
      - pattern: "useEffect\\([^,]+\\)"
        message: "Specify dependencies array in useEffect"

      - pattern: "useState\\([^)]*\\).*useState\\([^)]*\\)"
        message: "Consider combining related state variables"

      - pattern: "React\\.memo\\(.*\\)"
        message: "Ensure React.memo is used appropriately for performance"

  - type: suggest
    message: |
      React Best Practices:
      - Use functional components with hooks
      - Implement proper memoization
      - Follow the Rules of Hooks
      - Use TypeScript for prop types
      - Consider custom hooks for reusable logic

metadata:
  priority: high
  version: 1.0
</rule> 

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
