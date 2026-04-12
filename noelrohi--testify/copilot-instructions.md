## testify

> Follow these guidelines when creating React components within the project.

# Component Definition Guidelines

Follow these guidelines when creating React components within the project.

## General

-   **Function Components:** Define all components as Function Components.
    ```typescript
    // Preferred
    export function MyComponent(props: MyComponentProps) {
      // ...
    }

    // Also acceptable
    export const MyComponent = (props: MyComponentProps) => {
      // ...
    }
    ```
-   **TypeScript:** Use TypeScript for defining props and state.
    ```typescript
    interface MyComponentProps {
      title: string;
      count?: number;
    }

    export function MyComponent({ title, count = 0 }: MyComponentProps) {
      // ...
    }
    ```
-   **File Structure:** Keep component logic within its own file (e.g., `[MyComponent.tsx](mdc:src/components/my-component.tsx)`). For complex components, consider co-locating related hooks or types.

## Component Types

-   **Server Components:** These are the default in the Next.js App Router. Use them for fetching data directly on the server and rendering static or server-generated content. Avoid using hooks like `useState` or `useEffect`.
    -   Data fetching: Use the server TRPC client (`[api](mdc:src/trpc/server.ts)`). See `[src/app/(protected)/[spaceId]/page.tsx](mdc:src/app/(protected)/[spaceId]/page.tsx)`.
-   **Client Components:** Use the `"use client"` directive at the top of the file for components that require interactivity, browser APIs, or React hooks like `useState`, `useEffect`, `useContext`, etc.
    -   Data fetching: Use the client TRPC hooks from `[src/lib/trpc.ts](mdc:src/lib/trpc.ts)`. See `[src/app/spaces/[spaceId]/collector/page.client.tsx](mdc:src/app/spaces/[spaceId]/collector/page.client.tsx)`.
    -   Keep client components as small as possible ("leaf" components) to minimize the client-side JavaScript bundle.

## State and Logic

-   **State:** Use `useState` for simple local component state.
-   **Complex State:** Consider `useReducer` or state management libraries if state logic becomes complex or needs to be shared.
-   **Side Effects:** Use `useEffect` for side effects like API calls (in client components), subscriptions, or interacting with browser APIs.
-   **Data Fetching (Client):** Use the TanStack Query hooks provided by our TRPC setup (`useSuspenseQuery`, `useMutation`, etc.) for data fetching, caching, and mutations. See `[src/app/spaces/[spaceId]/collector/page.client.tsx](mdc:src/app/spaces/[spaceId]/collector/page.client.tsx)`.
-   **Forms:** Use `react-hook-form` with `zod` for form management and validation, as demonstrated in `[src/app/spaces/[spaceId]/collector/page.client.tsx](mdc:src/app/spaces/[spaceId]/collector/page.client.tsx)`.

## Styling

-   **Tailwind CSS:** Use Tailwind CSS utility classes for styling.
-   **UI Primitives:** Leverage the pre-built components from `[src/components/ui](mdc:src/components/ui)` (based on shadcn/ui) for common UI elements like buttons, inputs, dialogs, etc. See `[src/app/spaces/[spaceId]/collector/page.client.tsx](mdc:src/app/spaces/[spaceId]/collector/page.client.tsx)` for examples.

## Imports

-   **Absolute Paths:** Use absolute imports configured with `@/*` (e.g., `import { Button } from '@/components/ui/button';`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noelrohi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/noelrohi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
