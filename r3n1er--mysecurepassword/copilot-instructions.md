## architectural-guidelines-for-next-js-boilerplate

> description: Architectural guidelines for Next.js boilerplate



---
description: Architectural guidelines for Next.js boilerplate
globs: ["app/**", "lib/**", "components/**"]
alwaysApply: true
---

# Architectural Guidelines for Next.js Boilerplate

## Overview
Maintain a modular, scalable, and maintainable codebase for the Next.js boilerplate, inspired by best practices from Shipfast and Next.js Boilerplate.

## File Structure
```
my-boilerplate/
├── app/              # App Router routes
│   ├── (main)/       # Main application
│   ├── (landing)/    # Public landing page
│   └── (docs)/       # Nextra documentation
├── components/       # Reusable Shadcn components
├── lib/              # Service configurations (Supabase, Mailgun, etc.)
├── blog/             # Blog module
├── tests/            # Unit and integration tests
├── public/           # Static assets
├── .cursor/rules/    # Cursor rules
└── .env.example      # Environment variable template
```

## Coding Standards
- Use **TypeScript** with strict mode enabled (`tsconfig.json`).
- Avoid `any`; use precise types/interfaces.
- Use functional components with `React.FC`.
- Name directories in **lowercase-with-dashes** (e.g., `user-profile`).
- Use **camelCase** for variables/functions, **PascalCase** for components/interfaces.

## Modularity
- Encapsulate service logic in `lib/` (e.g., `lib/supabaseBrowserClient.ts`).
- Create reusable hooks for logic (e.g., `useAuth`, `useFormValidation`).
- Use **Zustand** for global state management, avoiding excessive context.

## Error Handling
- Use guard clauses for early returns.
- Provide user-friendly error messages.
- Log errors to console in development, integrate with Sentry in production.

## Best Practices
- Follow **DRY** principles; extract duplicate logic to helpers/hooks.
- Ensure all changes are backward-compatible.
- Document public APIs and components in `/docs`.

---
> Source: [R3n1er/mysecurepassword](https://github.com/R3n1er/mysecurepassword) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
