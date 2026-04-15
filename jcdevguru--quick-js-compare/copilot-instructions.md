## quick-js-compare

> - "Use interfaces over types for consistency and extendability."

# .cursorrules

# General TypeScript Code Style
typescript:
  - "Use interfaces over types for consistency and extendability."
  - "Avoid using 'any'; strive for strong typing wherever possible."
  - "Declare variable types explicitly (avoid implicit 'any')."
  - "Enable strict mode in tsconfig.json."

# Naming conventions
naming:
  classes: "Use PascalCase for classes and interfaces."
  variables: "Use camelCase for variable and function names."
  files: "Use kebab-case for filenames."
  folders: "Use lowercase with dashes for directories."

# Structure and Organization
structuring:
  files: 
    - "Group related content: components, interfaces, helper functions, and tests."
    - "Ensure each file has a clear purpose (e.g., Utilities, Comparison Logic)."
  exports:
    - "Favor named exports for functions and constants."

# Performance and Optimization
performance:
  - "Optimize function performance by dynamically generating them based on options."
  - "Leverage modularization to avoid redundancy."
  - "Utilize tree-shaking to ensure unused code is not included in the final bundle."

# Error Handling and Validation
error_handling:
  - "Use early returns for error conditions to prevent deeply nested logic."
  - "Implement comprehensive error logging and user-friendly error messages."

# Testing
testing:
  - "Use Jest for all testing requirements."
  - "Follow Arrange-Act-Assert pattern for writing tests."
  - "Ensure coverage for all public functions, focusing on the main object comparison logic."

# Specific Instructions for Comparison Logic
comparison_logic:
  - "Design main Comparison class to manage comparison logic dynamically."

# Configuration and Options Management
configurations:
  - "Base dynamic function generation on configuration options specified in README.md."

# Documentation
documentation:
  - "Comment your code extensively, especially the dynamically generated parts."
  - "Ensure all public methods include JSDoc comments detailing their purpose and parameters."

# Continuous Integration
ci:
  - "Integrate with CI/CD to automate testing and ensure code quality."
  - "Utilize linter rules to maintain consistent code style across the team."

# GUI/CLI
cli:
  - "Consider adding a command-line interface for configuration management."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jcdevguru) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
