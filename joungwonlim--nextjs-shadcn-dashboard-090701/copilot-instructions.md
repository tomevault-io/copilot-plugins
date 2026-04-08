## nextjs-shadcn-dashboard-090701

> This is a Next.js 15 project bootstrapped with `create-next-app`. It uses TypeScript, Tailwind CSS, and Turbopack for fast development. The UI is built with a design system based on Radix UI and `shadcn/ui` components.

# GEMINI.md

## Project Overview

This is a Next.js 15 project bootstrapped with `create-next-app`. It uses TypeScript, Tailwind CSS, and Turbopack for fast development. The UI is built with a design system based on Radix UI and `shadcn/ui` components.

The project is structured as a standard Next.js application with the App Router. The main application code is in the `app` directory, and reusable components are in the `components` directory.

## Building and Running

To get started, install the dependencies using `pnpm`:

```bash
pnpm install
```

Then, run the development server:

```bash
pnpm dev
```

This will start the development server on [http://localhost:3000](http://localhost:3000).

To build the project for production, run:

```bash
pnpm build
```

To run the production server, use:

```bash
pnpm start
```

To lint the code, run:

```bash
pnpm lint
```

## Development Conventions

*   **Components:** Components are organized in the `components` directory. The `ui` subdirectory contains the base UI components from `shadcn/ui`, while other files in the `components` directory are higher-level components that use the base components.
*   **Styling:** The project uses Tailwind CSS for styling. Utility classes are used directly in the components. The `tailwind.config.ts` file contains the Tailwind CSS configuration.
*   **Data:** The `app-sidebar.tsx` component uses sample data, which is a good starting point for understanding the data structures the application expects. In a real application, this data would likely come from an API.
*   **Linting:** The project uses ESLint for linting. The configuration is in the `eslint.config.mjs` file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joungwonlim)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joungwonlim)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
