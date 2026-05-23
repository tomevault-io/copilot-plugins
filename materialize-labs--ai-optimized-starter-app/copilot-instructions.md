## general

> Follow these rules for all requests.

# Project Guidelines

These guidelines serve as the foundation for our application development. Adhere to these principles to ensure consistency and quality across the codebase.

## Core Principles

- **Complete Implementation**: Write thorough, complete code for every feature
- **Consistency**: Follow established patterns throughout the codebase
- **Quality**: Focus on writing clean, maintainable, and tested code
- **Documentation**: Include appropriate comments and documentation

## Tech Stack Overview

Our application uses a modern, full-stack architecture:

- **Frontend**: 
  - Next.js with App Router
  - Tailwind CSS for styling
  - Shadcn UI component library
  - Framer Motion for animations

- **Backend**:
  - PostgreSQL via Supabase
  - Drizzle ORM for type-safe database access
  - Next.js Server Actions for API layer

- **Infrastructure**:
  - Authentication via Clerk
  - Deployment on Vercel

## Project Structure

```
/
├── actions/             # Server actions
│   ├── db/              # Database-related actions
│   └── ...              # Other action categories
├── app/                 # Next.js app router
│   ├── api/             # API routes
│   └── [route]/         # App routes
│       ├── _components/ # Route-specific components
│       ├── layout.tsx   # Route layout
│       └── page.tsx     # Route page
├── components/          # Shared components
│   ├── ui/              # UI components
│   └── utilities/       # Utility components
├── db/                  # Database
│   └── schema/          # Database schemas
├── lib/                 # Library code
│   └── hooks/           # Custom hooks
├── prompts/             # Prompt files
├── public/              # Static assets
└── types/               # Type definitions
```

## General Coding Standards

### File & Folder Naming

- Use kebab-case for all files and folders (e.g., `user-profile.tsx`)
- Component files should match their component names in kebab-case

### Import Conventions

- Use `@/` alias for imports from the app root
  ```tsx
  // Correct
  import { Button } from "@/components/ui/button"
  
  // Incorrect
  import { Button } from "../../components/ui/button"
  ```
- Group imports logically (React/Next.js, third-party, internal)

### Component Guidelines

- Don't modify shadcn components unless explicitly required
- Use appropriate HTML semantics in components

## Environment Variables

- Store all environment variables in `.env.local`
- Update `.env.example` when adding new variables
- Use `NEXT_PUBLIC_` prefix only for variables needed in the browser
- Access environment variables via `process.env.VARIABLE_NAME`

## Type System

### Type Definitions

- Place all types in the `types/` directory
- Prefer interfaces over type aliases
- Reference DB types from `@/db/schema` (e.g., `SelectContact`)

### Type Examples

```ts
// types/actions.ts
export type ActionState<T> =
  | { isSuccess: true; message: string; data: T }
  | { isSuccess: false; message: string; data?: never }
```

## Context Tags

Look for `<ai_context>` tags in the code - these provide important contextual information to help understand the codebase structure and functionality.

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
