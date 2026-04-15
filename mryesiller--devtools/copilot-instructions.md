## devtools

> This is a Next.js application that provides a collection of developer tools. The project is built with TypeScript, Tailwind CSS, and shadcn/ui.

# Project Overview

This is a Next.js application that provides a collection of developer tools. The project is built with TypeScript, Tailwind CSS, and shadcn/ui.

# Building and Running

To get started with the project, run the following commands:

```bash
npm install
npm run dev
```

This will start the development server at `http://localhost:3000`.

To build the project for production, use:

```bash
npm run build
```

To run the production server, use:

```bash
npm start
```

To run tests, use:

```bash
npm test
```

# Development Conventions

## Adding a New Tool

To add a new tool to the collection, follow these steps:

1.  **Metadata:** Create a new `.json` file in the `prompts/` directory (e.g., `prompts/uuid-generator.json`). This file defines the tool's metadata for the home page listing. It should contain:
    ```json
    {
      "name": "UUID Generator", // Display name on the home page card
      "description": "Generate different versions of UUIDs", // Description on the home page card
      "href": "/uuid-generator" // URL path for the tool (must match directory name)
    }
    ```
2.  **Routing:** Create a new directory inside `app/(home)/` with a name matching the `href` value (without the leading slash), e.g., `app/(home)/uuid-generator/`.
3.  **Page Component:** Inside the new tool directory, create a `page.tsx` file. This is a simple wrapper component that imports and renders your main tool component. Example:
    ```typescript
    // app/(home)/uuid-generator/page.tsx
    import { UuidGenerator } from "./UuidGenerator";
    // Optional: import { metadata } from "./metadata";

    // Optional: export { metadata };

    export default function UuidGeneratorPage() {
      return <UuidGenerator />;
    }
    ```
4.  **(Optional) Metadata Export:** For better SEO, create a `metadata.ts` or `.tsx` file in the tool directory and export a `metadata` object following Next.js conventions.
5.  **Tool Component:** Create your main tool component file (e.g., `app/(home)/uuid-generator/UuidGenerator.tsx`).
    *   Add the `"use client";` directive at the top.
    *   Implement the tool's UI using components from `@/components/ui/` (shadcn/ui).
    *   Use React hooks (`useState`, `useCallback`, `useEffect`, `useTransition`) for state management and logic.
    *   Use the `useToast` hook and `<Toaster />` component for user feedback (like copy actions).
    *   Ensure the component is responsive and accessible (use appropriate ARIA attributes, semantic HTML, etc.).
6.  **Home Page:** The home page automatically picks up the new tool from the `prompts/` directory. No manual update is needed there.

## Code Style

The project uses ESLint and Prettier for code linting and formatting. Please ensure that your code adheres to the project's coding style by running `npm run lint` before committing your changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mryesiller)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mryesiller)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
