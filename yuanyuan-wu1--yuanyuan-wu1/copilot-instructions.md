## yuanyuan-wu1

> *.ts linguist-language=TypeScript

# Next.js Portfolio Project Cursor Rules

# TypeScript/JavaScript files
*.ts linguist-language=TypeScript
*.tsx linguist-language=TypeScript
*.js linguist-language=JavaScript
*.jsx linguist-language=JavaScript

# Ignore build files
.next/* linguist-generated
out/* linguist-generated
node_modules/* linguist-vendored

# Custom rules for portfolio content
src/utils/portfolio.ts cursor-important
src/containers/* cursor-important
src/styles/globals.css cursor-important

# Documentation
*.md cursor-documentation

# Configuration files
*.json cursor-configuration
*.config.js cursor-configuration
.env* cursor-configuration

# Assets
public/* cursor-assets

# Testing
**/*.test.ts cursor-test
**/*.spec.ts cursor-test

# Animations
src/animations/* cursor-animation

# Rules for AI
ai-rules:
  # Project specific conventions
  - Follow TailwindCSS class naming conventions
  - Use TypeScript strict mode
  - Follow Next.js 13+ best practices

  # Code style
  - Use functional components with hooks
  - Prefer const over let
  - Use arrow functions
  - Use async/await over promises

  # Component structure
  - Keep components small and focused
  - Extract reusable logic to custom hooks
  - Place shared types in separate files

  # Performance
  - Use React.memo for expensive components
  - Implement proper loading states
  - Use Next.js Image component for images

  # Animation guidelines
  - Use Framer Motion for animations
  - Keep animations subtle and performant

  # Testing
  - Write unit tests for utilities
  - Test component rendering
  - Mock external services

  # Documentation
  - Document complex functions
  - Add JSDoc comments for components
  - Keep README.md updated

  # State management
  - Use React Context for global state
  - Prefer local state when possible
  - Implement proper error boundaries

cursor-settings:
  editor:
    formatOnSave: true
    tabSize: 2
    insertSpaces: true
    defaultFormatter: "prettier"

  typescript:
    strict: true
    checkJs: true

  tailwind:
    validate: true
    suggestions: true

  prettier:
    singleQuote: true
    semi: true
    trailingComma: "es5"

  eslint:
    autofix: true

  git:
    enabled: true
    autoCommit: false

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yuanyuan-Wu1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
