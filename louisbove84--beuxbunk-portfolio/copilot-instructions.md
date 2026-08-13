## beuxbunk-portfolio

> - Use functional and declarative programming; avoid classes

# =======================================
# Minimal Cursor Rules - Next.js + React
# =======================================

# ---- Technology Focus ----
technologies:
  - JavaScript
  - TypeScript
  - React
  - Next.js
  - Zustand
  - Tailwind
  - Shadcn UI
  - Radix UI
  - Stylus

# ---- Code Style ----
code_style:
  - Use functional and declarative programming; avoid classes
  - Descriptive variable/function names (camelCase)
  - React components in PascalCase
  - 2-space indentation, single quotes, no semicolons unless needed
  - Keep functions small and single-purpose
  - Avoid deeply nested logic

# ---- React Best Practices ----
react:
  - Follow Rules of Hooks; use useState, useEffect, useContext, useReducer, useMemo, useCallback properly
  - Prefer composition over inheritance
  - Use React.memo and useCallback where appropriate
  - Controlled components for forms; error boundaries for robustness
  - Use Suspense and dynamic imports for code splitting

# ---- State Management ----
state_management:
  - Use Zustand for global state
  - Lift state for shared state; context for intermediate state

# ---- Styling ----
ui_styling:
  - Tailwind for utility classes; Stylus modules for component-specific styles
  - Component-specific styles: .module.styl, camelCase class names
  - Import Stylus modules: import styles from './ComponentName.module.styl'
  - Minimal global styles

# ---- File Structure ----
file_structure:
  - Organize by feature/module
  - Components in components/, reusable code in lib/
  - Config files in config/ when needed

# ---- Performance ----
performance:
  - Favor server components and SSR; limit 'use client'
  - Lazy-load non-critical components
  - Optimize images (WebP, sizes, lazy-loading)
  - Route-based code splitting

# ---- Testing ----
testing:
  - Unit tests with Jest + React Testing Library
  - Meaningful test names and clear assertions
  - Run tests before committing

# ---- Security ----
security:
  - Never commit secrets
  - Sanitize all user inputs
  - Avoid outdated dependencies

# ---- Commit Messages ----
commit_messages:
  - Subject <= 50 chars, imperative mood
  - No trailing period
  - Body <= 72 chars per line if needed

# ---- Collaboration ----
collaboration:
  - Pull latest changes before work
  - Focus branches on single feature/topic
  - Write clear PR descriptions
  - Resolve merge conflicts carefully and test

# ---- Documentation ----
documentation:
  - Keep concise, meaningful inline comments
  - Public functions/components should have docstrings
  - Update README.md for setup and usage

---
> Source: [louisbove84/beuxbunk-portfolio](https://github.com/louisbove84/beuxbunk-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
