## javascript-standards

> Standards for JavaScript development in Drupal

# JavaScript Development Standards

<rule>
name: javascript_standards
description: Enforce JavaScript development standards for Drupal

actions:
  - type: enforce
    conditions:
      - pattern: "\\(function\\s*\\(\\$\\)\\s*\\{[^}]*\\}\\)\\s*\\(jQuery\\)"
        message: "Use Drupal behaviors instead of IIFE"

      - pattern: "\\$\\('[^']*'\\)"
        message: "Cache jQuery selectors for better performance"

      - pattern: "\\.ajax\\(\\{[^}]*success:"
        message: "Implement proper error handling for AJAX calls"

      - pattern: "var\\s+"
        message: "Use const or let instead of var (ES6+)"

  - type: suggest
    message: |
      JavaScript Best Practices:
      - Use Drupal behaviors for all JavaScript
      - Implement proper error handling for AJAX
      - Cache jQuery selectors
      - Use ES6+ features
      - Add proper documentation
      - Follow Drupal JavaScript coding standards
      - Use proper event delegation
      - Implement proper error handling
      - Use async/await for asynchronous operations
      - Follow proper module pattern

  - type: validate
    conditions:
      - pattern: "Drupal\\.behaviors\\.\\w+"
        message: "Implement JavaScript functionality using Drupal behaviors"

      - pattern: "/\\*\\*[^*]*\\*/"
        message: "Add JSDoc documentation for JavaScript functions"

metadata:
  priority: high
  version: 1.0
</rule>

---
> Source: [ivangrynenko/cursorrules](https://github.com/ivangrynenko/cursorrules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
