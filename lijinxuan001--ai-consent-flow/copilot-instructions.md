## ai-consent-flow

> This document provides context for the AI Consent Flow project.


# GEMINI.md - AI Consent Flow

This document provides context for the AI Consent Flow project.

## Project Overview

This is a web application built with Next.js (v15) and TypeScript. It serves as a platform for managing medical consent forms, featuring a multi-step user interface for patients. The project was initially created using [v0.app](https://v0.app).

The application guides users through the following steps:
1.  **Review:** View a summary of the consent document.
2.  **Q&A:** Ask questions about the procedure via an AI-powered system.
3.  **Consent:** Provide digital or verbal consent.
4.  **Completion:** Confirmation that the process is complete.

It also includes an admin dashboard for managing the consent process.

### Key Technologies

*   **Framework:** [Next.js](https://nextjs.org/) (React)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **UI Components:** [shadcn/ui](https://ui.shadcn.com/) (built on Radix UI)
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/)
*   **Icons:** [Lucide React](https://lucide.dev/)
*   **Package Manager:** The presence of `pnpm-lock.yaml` suggests `pnpm` is used.

## Building and Running

The following scripts are available in `package.json` and should be run with `pnpm`.

*   **Install dependencies:**
    ```bash
    pnpm install
    ```

*   **Run the development server:**
    ```bash
    pnpm dev
    ```
    The application will be available at [http://localhost:3000](http://localhost:3000).

*   **Build for production:**
    ```bash
    pnpm build
    ```

*   **Run the production server:**
    ```bash
    pnpm start
    ```

*   **Lint the code:**
    ```bash
    pnpm lint
    ```

## Development Conventions

*   **Structure:** The application follows the Next.js App Router structure.
    *   Pages are located in the `app/` directory.
    *   Reusable UI components are in `components/`.
    *   `shadcn/ui` components are located in `components/ui/`.
    *   Global styles are in `app/globals.css`.
*   **Path Aliases:** The project uses path aliases for easier imports (e.g., `@/components/*`). These are configured in `tsconfig.json`.
*   **Configuration:**
    *   `next.config.mjs`: Note that ESLint and TypeScript error checks are currently disabled during the build process (`ignoreDuringBuilds: true`, `ignoreBuildErrors: true`).
*   **Components:** The UI is built using `shadcn/ui`, which favors composition and utility-first styling. New components should follow this pattern.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lijinxuan001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lijinxuan001)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
