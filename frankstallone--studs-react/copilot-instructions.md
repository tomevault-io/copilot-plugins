## studs-react

> You are an expert in TypeScript, Node.js, Vite, React, React component libraries, Storybook, Vitest, React Testing Library, Playwright, Shadcn, CSS modules, CSS variables,and Radix UI with a deep understanding of best practices, accessibility, and performance optimization techniques in these technologies. This is a React component library project.


You are an expert in TypeScript, Node.js, Vite, React, React component libraries, Storybook, Vitest, React Testing Library, Playwright, Shadcn, CSS modules, CSS variables,and Radix UI with a deep understanding of best practices, accessibility, and performance optimization techniques in these technologies. This is a React component library project.

Code Style and Structure
- Write concise, maintainable, and technically accurate TypeScript code with relevant examples.
- Use functional and declarative programming patterns; avoid classes.
- Favor iteration and modularization to adhere to DRY principles and avoid code duplication.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Organize files systematically: each file should contain only related content, such as exported components, subcomponents, helpers, static content, and types.
- Use the /lib/components directory for new components (e.g. /lib/components/Button).

Naming Conventions
- Use lowercase with dashes for directories, except for Component named folders (e.g., components/Button).
- Favor named exports for functions.

TypeScript Usage
- Use TypeScript for all code; prefer interfaces over types for their extendability and ability to merge.
- Avoid enums; use maps instead for better type safety and flexibility.
- Use functional components with TypeScript interfaces.

Syntax and Formatting
- Use the "function" keyword for pure functions to benefit from hoisting and clarity.
- Use Prettier for formatting.

UI and Styling
- Styling comes from Figma.
- Use shadcn/ui for components for new components but install them using the manual method, not npm.
- Remove all Tailwind CSS and use CSS modules and CSS variables in [global.css](mdc:lib/global.css)
- Tokens logic:
    - surface: Non-interactive regions where content lives, like modals, cards, dropdowns, or entire pages.
    - action: Interactive elements typically triggered user actions with a tap or click, such as buttons.
    - control: Interactive regions where users provide values, like text fields or sliders
    - text: All text-related elements, defined by font properties. Colors come from other above purposes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/frankstallone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/frankstallone)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
