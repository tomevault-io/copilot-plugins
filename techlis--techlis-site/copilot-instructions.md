## techlis-site

> This document provides a comprehensive overview of the Techlis website project, designed to give the Gemini code assistant the context it needs to provide effective and relevant assistance.

# GEMINI Code Assistant Context

This document provides a comprehensive overview of the Techlis website project, designed to give the Gemini code assistant the context it needs to provide effective and relevant assistance.

## Project Overview

The project is a professional, luxury static landing page for an AI/software development startup named Techlis. It is built with a modern tech stack, including React, TypeScript, and Tailwind CSS, and is optimized for performance, SEO, and accessibility.

- **Framework**: Vite + React 19 + TypeScript
- **Styling**: Tailwind CSS v3.4 + Framer Motion for animations
- **UI Components**: Custom components built with Radix UI primitives
- **Routing**: React Router DOM v7
- **Testing**: Vitest + Testing Library
- **Deployment**: Netlify with CI/CD

The codebase is well-structured, with a clear separation of concerns. The `src` directory contains all the source code, organized into components, pages, libraries, styles, and content.

## Building and Running

The project uses `pnpm` as its package manager. The following scripts are available in `package.json`:

- `pnpm dev`: Starts the development server with hot reload at `http://localhost:3000`.
- `pnpm build`: Builds the project for production.
- `pnpm preview`: Previews the production build locally.
- `pnpm test`: Runs the test suite using Vitest.
- `pnpm lint`: Lints the codebase using ESLint.
- `pnpm format`: Formats the code using Prettier.
- `pnpm type-check`: Checks for TypeScript errors.

## Development Conventions

The project follows a set of development conventions to ensure code quality and consistency.

### Coding Style

- **TypeScript**: The project is written in TypeScript, and all new code should be as well.
- **ESLint**: The project uses ESLint to enforce a consistent coding style. The configuration can be found in `.eslintrc.json`. Key rules include:
  - `react/react-in-jsx-scope`: "off" (not required with the new JSX transform)
  - `@typescript-eslint/no-unused-vars`: "error"
  - `@typescript-eslint/explicit-function-return-type`: "warn"
  - `prefer-const`: "error"
  - `no-var`: "error"
- **Prettier**: The project uses Prettier to automatically format code. The configuration can be found in `.prettierrc`.
- **File Naming**: Components and pages should be named using PascalCase (e.g., `MyComponent.tsx`).

### Component Structure

- **Atomic Design**: The project follows an atomic design methodology, with components organized into `ui`, `layout`, `sections`, and `common` directories.
- **`ui`**: Contains basic UI elements like buttons, cards, and inputs.
- **`layout`**: Contains layout components like `Header`, `Footer`, and `Layout`.
- **`sections`**: Contains the main sections of the website, such as `Hero`, `Services`, and `About`.
- **`common`**: Contains reusable components that don't fit into the other categories.

### State Management

For a static site like this, complex state management libraries like Redux are not necessary. Component-level state managed with `useState` and `useReducer` is preferred.

### Testing

The project uses Vitest and the React Testing Library for testing. All new components and features should be accompanied by tests.

- **Unit Tests**: For individual components and functions.
- **Integration Tests**: For testing the interaction between multiple components.

## Customization

The website is designed to be easily customizable.

- **Content**: All the website's content (text, links, etc.) is stored in `src/lib/constants.ts` and the `src/content/` directory.
- **Styling**: The website's styling can be customized by editing `tailwind.config.ts` and `src/styles/globals.css`.
- **Components**: All components are located in the `src/components/` directory and can be easily modified.

By following these guidelines, the Gemini code assistant will be able to provide accurate, consistent, and helpful assistance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Techlis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Techlis)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
