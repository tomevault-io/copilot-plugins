## databreef-io

> This repository is optimized for AI-driven development. To maintain high efficiency and stay within API rate limits, all agents must adhere to the following file management rules.

# DataBreef Agent Instructions

This repository is optimized for AI-driven development. To maintain high efficiency and stay within API rate limits, all agents must adhere to the following file management rules.

## 📏 Source File Constraints

### 300-Line Limit
- **Rule**: No source code file (`.ts`, `.tsx`, `.css`) should exceed **300 lines**.
- **Action**: When a file reaches approximately 300 lines, you MUST split it into smaller, more focused modules.
- **Why**: 
  - Prevents context window bloat.
  - Reduces token usage per edit.
  - Minimizes the risk of rate limiting during large refactors.
  - Improves code readability and testability.

## 🏗 Modularization Strategy

1. **Components**: Split large components into sub-components in a local directory (e.g., `MyComponent/SubComponent.tsx`).
2. **Logic**: Move complex logic, data fetching, or transformations out of React components and into utility files or custom hooks.
3. **Styles**: Use scoped CSS Modules to prevent large, monolithic stylesheets.
4. **Types**: Move shared types to dedicated `types.ts` files within the feature directory.

## ⚓ The Metaphor

Maintain the ocean/reef metaphor in all new UI and documentation:
- **Reef/Source** = Database
- **Anchor** = Connect
- **Surface** = View/Generate
- **Dive/Brief** = Analysis/Dib

## 🔐 Security Standards

- **Read-Only**: Always apply `SET TRANSACTION READ ONLY` for introspection.
- **No Global Sessions**: Avoid session-level locks (`SET SESSION CHARACTERISTICS`) to prevent pooler leakage.
- **Encryption**: connection strings MUST be AES-256 encrypted at rest.

---
> Source: [DataBreef-io/DataBreef.io](https://github.com/DataBreef-io/DataBreef.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
