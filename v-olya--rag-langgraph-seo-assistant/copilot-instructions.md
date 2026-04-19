## rag-langgraph-seo-assistant

> description: This file provides coding guidelines for TypeScript projects.


description: This file provides coding guidelines for TypeScript projects.
__________________________________

Write modular, scalable and maintainable code following these rules:

- Use strict type checking
- Prefer type inference when the type is obvious
- Separate types and interfaces into their own files when they grow large
- Never use the `any` type; choose `unknown` when type is uncertain
- Never disable ESLint or TypeScript rules. Never!
- Do not add unnecessary comments; write self-explanatory code
- Use descriptive names for variables, functions, and types
- Avoid using hardcoded values; use constants or enums instead
- Please, no default exports except of required by a framework

- Keep components small and focused on a single responsibility
- Extract reusable logic into helper functions or custom hooks
- Ensure accessibility with semantic HTML and ARIA attributes
- Write unit tests for components and functions

When you need library documentation (for code generation, setup, configuration, etc.), use **Context7 MCP** to find the most relevant and up-to-date information.

Before responding with the generated code, perform type check and run linter. Double check if your code is DRY.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/v-olya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
