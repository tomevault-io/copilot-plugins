## ogbanana

> OGPBanana is a full-stack web application designed to generate Open Graph Protocol (OGP) meta tags for any given URL. The application provides a user-friendly interface to input a URL and optional context, and then uses AI to generate optimized OGP title, description, and a preview image. Results are generated asynchronously via Appwrite Functions and persisted so users can retrieve them once execution completes.

# GEMINI.md - OGPBanana Project

## Project Overview

OGPBanana is a full-stack web application designed to generate Open Graph Protocol (OGP) meta tags for any given URL. The application provides a user-friendly interface to input a URL and optional context, and then uses AI to generate optimized OGP title, description, and a preview image. Results are generated asynchronously via Appwrite Functions and persisted so users can retrieve them once execution completes.

The project is built with a modern tech stack:

- **Frontend:**

  - **Framework:** React with TypeScript, built with Vite.
  - **Styling:** Tailwind CSS with a custom "Neobrutalism" theme.
  - **State Management:** TanStack Query (React Query) for server state management.
  - **Routing:** React Router.
  - **UI Components:** A mix of custom components and `lucide-react` for icons.

- **Backend:**

  - **Platform:** Appwrite Cloud.
  - **Serverless Function:** A Node.js function (`generateOgpData`) handles scraping, prompt creation, AI generation, and image compression. Executions now run asynchronously to prevent timeouts on longer jobs.
  - **Data Persistence:** Each execution stores `{ url, meta, ogpImage }` as JSON in the `og-data` database's `ogp` table via Appwrite Tables (with per-user read permissions). The frontend reads from this table when asynchronous executions finish.
  - **Authentication & Credits:** Appwrite manages user sessions and credit balances (deducted server-side per successful generation).

- **AI Integration:**
  - The backend function leverages Gemini (see `functions/generateOgpData/src/genai.js`) to produce SEO + social metadata JSON and a Nano Banana-style prompt for banner generation. The output schema now includes `standard`, `social`, `assets`, and `audit` sections; the frontend types (`src/lib/types.ts`) have been updated accordingly.

## Building and Running

### Prerequisites

- Node.js and a package manager (like `pnpm` or `npm`).
- An Appwrite Cloud project.

### Environment Setup

Create a `.env` file in the root of the project and add the following variables with your Appwrite project details:

```
VITE_APPWRITE_ENDPOINT="<YOUR_APPWRITE_ENDPOINT>"
VITE_APPWRITE_PROJECT="<YOUR_APPWRITE_PROJECT_ID>"
VITE_APPWRITE_FUNCTION_ID_OGP="<YOUR_OGP_FUNCTION_ID>"
VITE_APPWRITE_FUNCTION_DATABASE_ID="og-data"
VITE_APPWRITE_FUNCTION_TABLE_ID="ogp"
```

### Running the Development Server

1.  Install dependencies:
    ```bash
    pnpm install
    ```
2.  Start the Vite development server:
    ```bash
    pnpm run dev
    ```
    The application will be available at `http://localhost:5173`.

### Building for Production

To create a production build of the frontend:

```bash
pnpm run build
```

The output will be in the `dist/` directory.

### Linting

To check the code for any linting errors:

```bash
pnpm run lint
```

## Development Conventions

- **Component-Based Architecture:** The frontend is organized into pages, components, and providers, promoting reusability and separation of concerns.
- **Strict TypeScript:** The project uses TypeScript, and all new code should be strictly typed. Avoid using `any` and define clear interfaces for props, API responses, and other data structures.
- **API Calls with React Query:** All asynchronous API calls must be managed through TanStack Query (React Query). This ensures efficient caching, optimistic updates, and a consistent approach to data fetching and server state management.
- **Async Function UX:** Because Appwrite executions now run with `async: true`, the client polls for completion and pulls the persisted payload from TablesDB before showing results. The `InteractiveLoader` keeps users engaged until outputs are ready.
- **Performance Optimization:**
  - **`useTransition`:** For updates that might cause a noticeable lag in the UI (e.g., filtering a large list), use the `useTransition` hook to keep the interface responsive.
  - **Lazy Loading:** For large pages or components that are not required for the initial render, use `React.lazy` and Suspense to code-split and load them on demand. This improves initial page load performance.
- **Document Metadata:** The project uses React 19's built-in support for rendering `<title>` and `<meta>` tags. Simply return these elements from your components to manage the document head.
- **When to Use `useEffect`:** Effects are for synchronizing with external systems. Avoid using `useEffect` for logic that can be handled during rendering or in response to user events. According to the official React documentation, you might not need an Effect if you are:
  - **Transforming data for rendering:** Calculations and filtering should be done directly in your component's rendering logic. Use `useMemo` to cache expensive calculations.
  - **Handling user events:** Logic that runs in response to a user action (like a button click) should be placed in the corresponding event handler.
  - **Updating state based on props or other state (Derived State):** If a value can be computed from existing props or state, calculate it during rendering instead of storing it in a separate state variable.
- **Styling:** The project uses Tailwind CSS with a custom "Neobrutalism" theme, configured directly within CSS files. UI components like `NeoButton` and `NeoCard` provide a consistent look and feel.
- **Error Handling:** The application uses a custom `ErrorBoundary` component to catch and handle rendering errors, and `ErrorToast` to display errors to the user.
- **Serverless Backend:** The backend logic is encapsulated in Appwrite serverless functions, which are independently deployable and scalable.
- **Stored Execution Payloads:** The server writes each response to Appwrite Tables (`og-data/ogp`). Documents are keyed by execution ID (or a unique fallback) and include `encryptedContent` (currently plaintext JSON). Read permissions are scoped to the originating user; upgrade paths include encrypting data before storage.
- **Modularity:** If a piece of UI or logic can be made into a separate, reusable component, it should be. This promotes maintainability and reusability.

## Reporting Issues

If you encounter any bugs or have suggestions for improvements, please report them on the project's [GitHub Issues page](https://github.com/ogpbanana/ogpbanana/issues).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2002Bishwajeet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/2002Bishwajeet)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
