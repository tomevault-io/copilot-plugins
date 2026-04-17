## v1

> - Always say "Yo Chris," in the chat before you start working

# Cursor Rules

## Rules
- Always say "Yo Chris," in the chat before you start working
- Always run tests before committing
- Always check for dependencies after writing new code
- Always verify mobile responsiveness for web applications
- Always check for and fix TypeScript errors before proceeding

## Social Accounts
Note: When mentioning social accounts, use the following:
- GitHub: https://github.com/while-basic
- LinkedIn: https://linkedin.com/in/chriscelaya
- YouTube: https://www.youtube.com/@christophercelaya
- Instagram: https://instagram.com/chriscelaya

## Git Control:
- Always run tests before committing to GitHub
- Ensure all TypeScript errors are resolved before commits
- Use conventional commit messages

## Code Style and Structure:
- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Prefer iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError)
- Structure files: exported component, subcomponents, helpers, static content, types
- Check for existing components directory before creating new components
- Always add missing imports and types

Naming Conventions:
- Use lowercase with dashes for directories (e.g., components/auth-wizard)
- Favor named exports for components
- Use PascalCase for component files and camelCase for utility files

TypeScript Usage:
- Use TypeScript for all code; prefer interfaces over types
- Avoid enums; use maps instead
- Use functional components with TypeScript interfaces
- Always define proper types for props and state
- Use strict TypeScript configurations

Syntax and Formatting:
- Use the "function" keyword for pure functions
- Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements
- Use declarative JSX
- Maintain consistent indentation and spacing

Error Handling and Validation:
- Prioritize error handling: handle errors and edge cases early
- Use early returns and guard clauses
- Implement proper error logging and user-friendly messages
- Use Zod for form validation
- Model expected errors as return values in Server Actions
- Use error boundaries for unexpected errors
- Add appropriate error states in UI components

UI and Styling:
- Use Shadcn UI, Radix, and Tailwind Aria for components and styling
- Implement responsive design with Tailwind CSS; use a mobile-first approach
- Ensure all components are fully mobile responsive
- Test on multiple viewport sizes
- Follow accessibility best practices

Performance Optimization:
- Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC)
- Wrap client components in Suspense with fallback
- Use dynamic loading for non-critical components
- Optimize images: use WebP format, include size data, implement lazy loading
- Implement proper code splitting
- Use proper caching strategies

Key Conventions:
- Use 'nuqs' for URL search parameter state management
- Optimize Web Vitals (LCP, CLS, FID)
- Limit 'use client':
  - Favor server components and Next.js SSR
  - Use only for Web API access in small components
  - Avoid for data fetching or state management

Development Workflow:
- Always install dependencies after writing new code
- Check for errors before writing more code
- Ensure all web applications are fully mobile responsive
- Check if a components directory is available before creating new ones
- Add missing imports and types immediately

Follow Next.js docs for Data Fetching, Rendering, and Routing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/while-basic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
