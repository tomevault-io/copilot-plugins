## best-practices

> 1. Core Architectural Principles


1. Core Architectural Principles
   Separation of Concerns (The "Server-Client" Boundary)

Pattern: Keep your data fetching logic in Server Components and your interactivity logic in Client Components.

Why: This separates "backend logic" (database calls, API secrets) from "frontend logic" (state, effects, browser APIs).

Example: Create a UserDashboard (Server Component) that fetches data and passes it as props to UserProfile (Client Component) for editing.

DRY (Don't Repeat Yourself) with Layouts

Pattern: Use layout.tsx files to define shared UI (navbars, sidebars, footers) for specific route segments.

Why: React doesn't re-render the layout when navigating between sibling pages, preserving state (like scroll position or search input) and reducing code duplication.

Logic Extraction (Custom Hooks)

Pattern: Never write complex useEffect or data manipulation logic directly inside your UI components. Extract them into custom hooks (e.g., useCart, useAuth).

Goal: Your component should just describe the UI, not calculate it.

Refactor: If a component is >200 lines, it usually does too much.

2. Next.js App Router Specifics
   Colocation over Centralization

Practice: Instead of a giant /components folder, place components inside the route folder they belong to if they are only used there.

Structure:

Plaintext

app/
├── dashboard/
│ ├── page.tsx
│ ├── \_components/ <-- Components specific to dashboard
│ │ ├── DashboardCard.tsx
│ │ └── AnalyticsChart.tsx
│ └── layout.tsx
Note: Use an underscore (\_components) to exclude the folder from the router.

Server Actions for Mutations

Practice: Replace traditional API Routes (pages/api/...) with Server Actions for form submissions and data mutations.

Why: It keeps the mutation logic right next to the UI that triggers it, offers full type safety, and works without JavaScript (progressive enhancement).

Route Groups for Organization

Practice: Use (folderName) syntax to organize files logically without affecting the URL path.

Example: app/(marketing)/page.tsx and app/(admin)/dashboard/page.tsx. This allows you to have different root layouts for "Marketing" pages vs. "Admin" pages.

3. Data & Performance
   Fetch Data Where It's Needed

Practice: Don't fetch all data in a parent and prop-drill it down 5 levels. Fetch data directly in the component that needs it (in Server Components). Next.js automatically deduplicates fetch requests, so calling the same API in multiple components is cheap/free.

Term: This is often called "Request Memoization."

Use Suspense for Streaming

Practice: Wrap slow-loading components (like a heavy chart or a feed) in a <Suspense> boundary with a fallback skeleton.

Why: This allows the rest of the page (header, sidebar) to load instantly while the heavy data streams in later, preventing the "blank white screen" effect.

Optimize Images & Fonts

Practice: Always use next/image instead of <img> to prevent Layout Shifts (CLS) and automatically serve WebP/AVIF formats. Use next/font to self-host Google fonts automatically (zero layout shift).

4. Code Quality & Security
   Zod for Validation (The "Parse, Don't Validate" Mantra)

Practice: Do not trust data coming from anywhere (API, user input, URL params). Use Zod schema validation to ensure the data shape is exactly what you expect before your component tries to render it.

Why: It prevents undefined errors crashing your app at runtime.

Strict TypeScript

Practice: Avoid any at all costs. Use interface or type for all props.

Tip: Use the generic PropsWithChildren type or React.ReactNode for components that accept children.

---
> Source: [JaimeenMakavana/agent-vis](https://github.com/JaimeenMakavana/agent-vis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
