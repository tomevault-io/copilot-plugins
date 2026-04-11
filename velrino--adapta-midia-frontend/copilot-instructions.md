## adapta-midia-frontend

> This document provides essential information for an AI agent to effectively work with the Adapta Midia codebase.

# AI Agent Context for Adapta Midia

This document provides essential information for an AI agent to effectively work with the Adapta Midia codebase.

## Project Overview

This is a [Next.js](https://nextjs.org/) project created using the `shadcn` package, as indicated in the `README.md`. It uses TypeScript for type safety and Tailwind CSS for styling. The project is configured with internationalization (i18n) using `typesafe-i18n`.

## Key Technologies

- **Framework:** [Next.js](https://nextjs.org/)
- **Language:** [TypeScript](https://www.typescriptlang.org/)
- **UI Components:** [shadcn/ui](https://ui.shadcn.com/), [Radix UI](https://www.radix-ui.com/)
- **Styling:** [Tailwind CSS](https://tailwindcss.com/)
- **Internationalization (i18n):** [typesafe-i18n](https://github.com/ivanhofer/typesafe-i18n)
- **Package Manager:** [pnpm](https://pnpm.io/)
- **Testing:** [Vitest](https://vitest.dev/)
- **Linting:** [ESLint](https://eslint.org/)
- **Formatting:** [Prettier](https://prettier.io/)
- **Git Hooks:** [Husky](https://typicode.github.io/husky/)

## Directory Structure

- `app/`: Contains the core application logic, following the Next.js App Router structure.
- `components/`: Houses reusable React components, organized by feature.
- `hooks/`: Custom React hooks for shared logic.
- `i18n/`: Internationalization files for different languages.
- `lib/`: Utility functions, business logic, and other shared code.
- `public/`: Static assets like images and fonts.
- `types/`: Global TypeScript type definitions.

## Development Workflow

### Prerequisites

Ensure you have [pnpm](https://pnpm.io/installation) installed.

### Installation

Install project dependencies with:

```sh
pnpm install
```

### Running the Development Server

To start the development server, run:

```sh
pnpm dev
```

### Building for Production

To create a production build, use:

```sh
pnpm build
```

### Running Tests

Execute the test suite with:

```sh
pnpm test
```

For an interactive test UI, run:

```sh
pnpm test:ui
```

### Linting and Formatting

To check for linting errors, run:

```sh
pnpm lint
```

To automatically format the code, use:

```sh
pnpm format
```

## Coding Style and Conventions

- Follow the existing coding style and conventions.
- Use the provided UI components from `shadcn/ui` and Radix UI.
- Ensure all new code is properly typed with TypeScript.
- Add tests for new features and bug fixes in the `lib/__tests__` directory.
- Keep the i18n files in `i18n/` updated with any new text.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/velrino)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/velrino)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
