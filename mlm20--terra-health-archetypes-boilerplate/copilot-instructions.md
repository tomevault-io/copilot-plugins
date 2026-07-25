## terra-health-archetypes-boilerplate

> - Use functional and declarative programming patterns; avoid classes.

# Project Coding Rules

- Use functional and declarative programming patterns; avoid classes.
- Prefer iteration and modularization over code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Use lowercase with dashes for directories (e.g., components/auth-wizard).
- Favor named exports for components.
- Handle errors and edge cases at the beginning of functions. Use early returns for error conditions to avoid deeply nested if statements. Place the happy path last for readability.

# Framework Specific Rules (Chakra UI)

- Prefer functional components with hooks.
- When using Chakra UI, follow best practices:
    - Use ChakraProvider at the root of your app.
    - Utilize Chakra UI components for consistent design.
    - Implement a custom theme for brand-specific styling.
    - Use responsive styles with the Chakra UI breakpoint system.
    - Leverage Chakra UI hooks for enhanced functionality.

# Additional Instructions

1. Use TypeScript for type safety with Chakra UI components.
2. Implement proper component composition using Chakra UI.
3. Utilize Chakra UI's built-in accessibility features.
4. Use the 'as' prop for semantic HTML rendering.
5. Implement dark mode using Chakra UI's color mode.
6. Use Chakra UI's layout components for responsive design.
7. Follow Chakra UI best practices for performance optimization.

---
> Source: [mlm20/terra-health-archetypes-boilerplate](https://github.com/mlm20/terra-health-archetypes-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
