## general

> Entire codebase

# Cursor Rules for LobbyAI

This document outlines the rules to guide Cursor's AI behavior within the LobbyAI project. These rules leverage Cursor's folder-specific configurations for granular control.

## I. General Rules

*   **Description:** Apply to all files in the project.
*   **File Pattern:** `**/*`
*   **Behavior:**
    *   Follow the existing code style and conventions.
    *   Prioritize readability and maintainability.
    *   Use TypeScript features effectively.
    *   Write clear and concise comments where necessary.
    *   Favor functional programming principles where appropriate.
    *   When introducing new dependencies make sure that it align with the rest of the stack and that it adds value to the product.
    *   When consuming data from the api, be type safe.
    *   When creating components, use the present UI components.

## II. React Components (`components/**/*`)

*   **Description:** Rules specific to React components within the `components` directory.
*   **File Pattern:** `components/**/*`
*   **Behavior:**
    *   Use functional components with hooks.
    *   Utilize the `cn` utility for conditional class names using Tailwind CSS.
    *   Structure components for reusability and composability.
    *   Follow the existing UI patterns in `/components/ui`.
    *   Keep components small and focused. Extract complex logic into custom hooks.
    *   For state management, leverage Zustand (`store/**/*`).
    *   Be aware of existing Themes when creating Components

## III. API Services (`services/**/*`)

*   **Description:** Rules for API service files in the `services` directory.
*   **File Pattern:** `services/**/*`
*   **Behavior:**
    *   Use `axios` for API requests.
    *   Handle errors gracefully with appropriate logging and error messages.
    *   Define TypeScript interfaces for request and response data.
    *   Use `baseUrl` for base URLs.
    *   Make sure that you consume data from the api in a type safe way, creating types or interfaces in `/types` when necessary.
    *   When logging errors, use the following convetion `console.error("Description of the error", error)`

## IV. Hooks (`hooks/**/*`)

*   **Description:** Rules specific to custom React hooks within the `hooks` directory.
*   **File Pattern:** `hooks/**/*`
*   **Behavior:**
    *   Follow the "use" naming convention for custom hooks.
    *   Keep hooks focused on specific logic (e.g., data fetching, authentication).
    *   Utilize `useSWR` for data fetching and caching.
    *   Return a consistent object with `data`, `error`, `isLoading`, and `mutate`.
    *   Add error handling and loging in all hooks.
    *   Make sure that you consume data from the api in a type safe way, creating types or interfaces in `/types` when necessary.
    *   When logging errors, use the following convetion `console.error("Description of the error", error)`

## V. Email Template Generation (`app/(Home)/components/AiCustomizationDialog.tsx`, `app/(Home)/components/ResultsTable.tsx`, `services/leads-api/email-generator.ts`)

*   **Description:** Rules relating to generating email templates.
*   **File Pattern:** `app/(Home)/components/AiCustomizationDialog.tsx, app/(Home)/components/ResultsTable.tsx, services/leads-api/email-generator.ts`
*   **Behavior:**
    *   Pay attention to how the AI email writer is customized using `AICustomizationDialog.tsx`.
    *   Ensure that company profile data is correctly utilized to personalize email templates.
    *   Verify that generated email templates are displayed correctly in `ResultsTable.tsx`.
    *   Verify that the templates follow the email guidelines from the company profile.
    *   When logging errors, use the following convetion `console.error("Description of the error", error)`
*   **Special Instructions:**
    *   When generating email templates, make sure the email guidelines from the company profile are present.
    *   Verify that the templates follow the email guidelines from the company profile.

## VI. Forms and Data Validation

*   **Description:** Rules pertaining to forms and data validation throughout the project.
*   **File Pattern:** `**/*`
*   **Behavior:**
    *   Employ `react-hook-form` and `zod` for form management and validation.
    *   Define schema validations in components like `CompanyProfileDialog.tsx`.
    *   Ensure user inputs are sanitized and validated against defined schemas.
    *   Present user-friendly error messages in the UI when validation fails.

## VII. Store and State Management

*   **Description:** Rules related to Zustand stores for global state management.
*   **File Pattern:** `store/**/*`
*   **Behavior:**
    *   Use Zustand for managing global application state (e.g., sidebar visibility, settings modal).
    *   Ensure state updates are efficient and avoid unnecessary re-renders.
    *   Create stores for specific domains to maintain separation of concerns.
    *   Provide clear and concise names for all state and actions within the store.

## VIII. UI Components

*   **Description:** Rules related to UI components like Avatar, Buttons, Modal, etc.
*   **File Pattern:** `components/ui/**/*`
*   **Behavior:**
    *   Components must be created using the `shadcn/ui` project for consistency
    *   Components should be created atomic as possible

## IX. Next.js Pages (`app/**/*`)

*   **Description:** Rules for Next.js pages located within the `app` directory.
*   **File Pattern:** `app/**/*`
*   **Behavior:**
    *   Use React Server Components (RSC) by default, unless client-side interactivity is required.
    *   Use `use client` to signal client-side components.
    *   When fetching data in server components, always do it within the api.
    *   Use Suspense for loading states in pages.
    *   Use `next/navigation` for routing and navigation.
    *   Always pass the variables to the components, do not consume the params inside the component.
    *   Favor the usage of Layout component

## X. Lib Directory (`lib/**/*`)

*   **Description:** Rules for reusable code.
*   **File Pattern:** `lib/**/*`
*   **Behavior:**
    *   When adding a new code there make sure it can be reused.
    *   Document the code added to this folder.

## XI. General TypeScript Guidelines

*   **Description:** General Typescript code guidelines.
*   **File Pattern:** `**/*`
*   **Behavior:**
    *   All Typescript code must be clean and follow Typescript best practices.
    *   Make sure that you consume data from the api in a type safe way, creating types or interfaces in `/types` when necessary.
    *   Check that any is not used, if it is, document why it was used.

## Automatic Attachment

*   These rules should be automatically attached when working on files matching the specified patterns.

## Notes

*   This document is a living document and will be updated as the project evolves.
*   When in doubt, ask a team member for clarification.

---
> Source: [Mandy9943/lobby-frontend2](https://github.com/Mandy9943/lobby-frontend2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
