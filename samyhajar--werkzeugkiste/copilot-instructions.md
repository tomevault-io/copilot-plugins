## werkzeugkiste

> I am an AI assistant helping with the development, testing, and maintenance of this project. I MUST ALWAYS follow these rules without exections!

# Agent Instructions

I am an AI assistant helping with the development, testing, and maintenance of this project. I MUST ALWAYS follow these rules without exections!

# General Instructions

- I MUST ONLY use Tailwind CSS v4 for styling - no custom CSS or other styling libraries.
- I MUST follow mobile-first design approach - all components MUST be fully responsive across all device sizes.
- I MUST NEVER start the development server as the user has it running in the background.
- I MUST ONLY make changes explicitly requested by the user. If I think additional changes would be beneficial, I MUST ask for confirmation first.
- I MUST ensure all code is production-ready, properly tested, and follows project conventions.
- I MUST accurately implement designs from Figma with pixel-perfect precision and proper responsive behavior.
- I MUST use Perplexity search proactively in these specific situations:
  - Before implementing any feature to research current best practices
  - When working with external APIs or libraries to verify syntax and usage patterns
  - When troubleshooting errors or unexpected behavior
  - When evaluating architectural decisions to ensure industry standards
  - To research security considerations for sensitive operations
  - When suggesting improvements to existing code or documentation
  - To verify compatibility with different browsers or platforms

# MANDATORY SELF-CHECK BEFORE EVERY RESPONSE

Before submitting any response that involves code, I MUST verify:

1. That I am ONLY using Tailwind CSS v4 for styling (no custom CSS)
2. That all code follows mobile-first responsive design principles
3. That I'm not suggesting launching the development server
4. That I'm only making changes explicitly requested by the user
5. For Supabase code: I am ONLY using approved patterns (detailed in Supabase section)
6. That UI components follow accessibility best practices
7. That my code follows the project's file organization structure
8. That TypeScript types are properly used with no 'any' types
9. That I've kept components under 200 lines with single responsibilities
10. That I ALWAYS follow DRY (Don't Repeat Yourself) principles in all code
11. That I'm using the @theme directive properly for design tokens instead of configuration files
12. That I've created separate pages with distinct URLs for each major phase of a workflow (e.g., separate pages for upload forms and result displays)
13. That I've implemented immediate visual feedback for all user interactions, especially for processes that take time (loading states, progress indicators, success/error messages)
14. That I've accurately implemented Figma designs with proper design tokens, spacing, typography, and component structure

# Tailwind CSS v4 Implementation

## Setup and Configuration

- I MUST understand that Tailwind CSS v4 uses the @theme directive in CSS to define theme variables, rather than configuration in JavaScript/TypeScript files.
- I MUST ensure that project has the correct dependencies:
  - tailwindcss (the core package)
  - @tailwindcss/postcss (for PostCSS integration)
  - postcss
- I MUST check that the CSS file imports Tailwind correctly using `@import "tailwindcss";` at the top of the main CSS file.
- I MUST confirm the postcss.config.mjs file includes `"@tailwindcss/postcss": {}` in its plugins.
- I MUST NEVER modify the compiled CSS output directly.

## Theme Variables and the @theme Directive

- I MUST define all design tokens using the @theme directive in the CSS file:
  ```css
  @import 'tailwindcss';
  @theme {
    --color-brand-primary: #3b82f6;
    --color-brand-secondary: #1e40af;
    /* Add all other design tokens here */
  }
  ```
- I MUST understand that theme variables are special CSS variables with these properties:
  - They MUST be defined at the top level (not nested inside other selectors)
  - They generate corresponding utility classes automatically (e.g., `--color-mint-500` creates `bg-mint-500`, `text-mint-500`, etc.)
  - They are also available as regular CSS variables for use in arbitrary values
- I MUST follow these variable naming conventions to map to the correct utility classes:
  - `--color-*`: Creates text, background, border, and other color utilities
  - `--font-*`: Creates font-family utilities
  - `--breakpoint-*`: Creates responsive variants (e.g., `--breakpoint-3xl: 120rem;` creates the `3xl:` breakpoint)
  - And other namespaces as defined in the Tailwind CSS documentation

## @theme vs. :root Variables

- I MUST use @theme for variables that should generate utility classes (design tokens)
- I SHOULD use regular :root CSS variables for values that don't need corresponding utility classes
- I MUST understand that @theme variables automatically create CSS variables that can be used with var() syntax

# User Experience and Accessibility (UX & a11y)

## Responsive Design & UI

- I MUST build responsive interfaces that work across modern browsers and devices.
- I MUST use Tailwind CSS v4 for consistent spacing and responsive layouts.
- I MUST ensure interactive elements are easily tappable on mobile (minimum 44x44px touch targets) and keyboard-navigable on desktop.
- I MUST provide visual feedback for user actions (loading states, disabled buttons, toast notifications).
- I MUST implement immediate visual feedback for ALL user interactions:
  - Show loading spinners/skeletons for ANY process that takes longer than 300ms
  - Display progress bars with percentage indicators for uploads and longer operations
  - Provide estimated time remaining for lengthy processes when possible
  - Disable interactive elements while actions are processing to prevent duplicate submissions
  - Animate transitions between loading and result states for a smoother experience
  - Use distinct visual cues for success, error, and in-progress states

## Accessibility Best Practices

- I MUST write semantic HTML with proper ARIA roles/attributes only when needed.
- I MUST use `<button>` for clickable actions, never `<div>`s; and MUST use `<form>` with `<label>` for inputs.
- I MUST include descriptive alt text on images and MUST use Next's Image component for optimization.
- I MUST implement focus management (logical tab order, focus trapping in modals, focus return).
- I MUST meet WCAG 2.1 AA guidelines for color contrast, text scaling, and keyboard navigation.
- I SHOULD configure eslint-plugin-jsx-a11y and run automated tests with axe-core.

## State Management & Navigation

- I MUST provide consistent UI with reusable styled components for common patterns.
- I MUST use URL state for all important UI states to enable deep linking and browser history.
- I MUST preserve form input on navigation and use HTML5 validation for instant feedback.
- I SHOULD leverage Next.js Route Announcements for screen reader navigation.

# Code Structure and Organization

## File Organization

- I MUST organize the code EXACTLY like this:
  - `/src/app`: Contains the application routes and page components using Next.js App Router
    - `/dashboard`: Main user area after authentication
    - `/auth`, `/login`, `/signup`: Authentication-related pages
    - `/api`: Server-side API endpoints
  - `/src/components`: Reusable UI components
    - `/dashboard`: Dashboard-specific components
    - `/ui`: UI components for consistent design
    - `/shared`: Common components used across the application
  - `/src/lib`: Core utilities and services
    - `/supabase`: Supabase client configuration and helpers
    - `/utils`: General utility functions
  - `/src/hooks/*`: Custom hooks for stateful logic and business rules
  - `/src/services/*`: API calls, data processing, and external service integration
  - `/src/types/*`: TypeScript type definitions
  - `/src/contexts/*`: React Context definitions for global state
  - `/supabase`: Supabase configuration files
    - `/migrations`: Database migrations
    - `/functions`: Edge and database functions

## Component Architecture and Organization

- I MUST organize components hierarchically with clear responsibilities:

  - Every component MUST have a single, clearly defined responsibility
  - Every page component MUST have a singular purpose corresponding to a specific user flow or application state
  - Page components (in page.tsx files) MUST be minimal and primarily focused on composition
  - Page components MUST NOT contain implementation details ("meat") but rather delegate to section components
  - Page components MUST be limited to layout, data fetching coordination, and section composition (50-100 lines maximum)
  - Each logical section of a page MUST be its own component (e.g., Header, Hero, Features, Testimonials, Footer)
  - Each section component MUST be further broken down into smaller, focused subcomponents
  - Section components MUST be in their own files in an organized directory structure

- I MUST follow these strict size and complexity limits:

  - Components MUST be kept under 200 lines of code (including imports and exports)
  - Component sections larger than 30 lines MUST be extracted into subcomponents
  - Components exceeding 150 lines MUST be split into smaller components
  - Components with too many props (>7) MUST be split or use composition
  - Component nesting MUST be limited to 3 levels maximum to prevent prop drilling

- I MUST organize related components hierarchically in dedicated directories:

  - Components MUST be organized reflecting their hierarchical relationship
  - For example: `/components/dashboard/ActivityFeed/ActivityItem.tsx`
  - Section components MUST be in their own directories with related subcomponents
  - Closely related components MUST be grouped in the same directory
  - Complex features (like a dashboard) MUST have their own directory containing all their specific components

- I MUST rigorously apply the "Single Responsibility Principle":

  - Each component MUST do exactly one thing and do it well
  - I MUST extract any code that handles a distinct visual or functional concern into its own component
  - I MUST create dedicated components for repeated patterns even if they appear only twice
  - I MUST NOT combine multiple logical phases of a workflow (like upload and results display) in a single page component

- I MUST ensure all components have clear interfaces:

  - Props MUST be well-defined with specific types and purpose
  - Props MUST be documented with JSDoc comments for non-obvious usage
  - Components MUST explicitly declare all dependencies they need to function

- I MUST follow DRY (Don't Repeat Yourself) principles rigorously by:

  - Creating reusable components for UI patterns that appear multiple times
  - Extracting repeated logic into custom hooks
  - Using utility functions for common operations
  - Leveraging higher-order components and render props when appropriate
  - Creating shared configuration constants instead of hardcoding the same values

- I MUST create separate component files for UI sections that:

  - Have their own state
  - Are reused in multiple places
  - Contain more than 50 lines of code

- I MUST use React Context for state shared across multiple components in different branches

- I MUST prefer composition over inheritance or complex nesting

## Separation of Concerns

- UI Components MUST:

  - Focus exclusively on rendering and user interaction
  - Use props for data and callbacks for events
  - Avoid containing business logic
  - Be written with mobile-first responsive design

- Custom Hooks MUST:

  - Encapsulate all state management
  - Handle complex business logic
  - Manage side effects
  - Be well-typed with TypeScript
  - Be unit-testable in isolation

- Service functions MUST:

  - Handle all external API calls
  - Manage data transformation
  - Encapsulate database operations
  - Use proper error handling

- Utility functions MUST:
  - Be pure functions without side effects
  - Have clear input/output contracts
  - Be unit-testable in isolation
  - Reside in appropriate utility files

## Prohibited Practices

- I MUST NEVER include in UI component files:

  - Direct API calls (use service files)
  - Complex state management logic (use custom hooks)
  - Utility functions (use utility files)
  - Database operations (use service files)
  - Media processing logic (use utility files)

- I MUST NEVER define:
  - Functions inside render methods that could be defined at module level
  - Complex event handlers (>10 lines) within components (move to hooks)
  - Anonymous functions with complex logic in JSX
  - Large, monolithic components (>200 lines)
  - Duplicate code that violates DRY principles, including:
    - Copy-pasted functions or components with minor variations
    - Repeated conditional logic that could be abstracted
    - Multiple implementations of the same business logic
    - Hardcoded values that should be constants or configuration
    - Redundant type definitions or interfaces

# Documentation Expectations

- I MUST add JSDoc comments for exported functions and types
- I MUST include brief explanations for complex logic
- I MUST document expected behaviors and edge cases in test files
- I MUST clearly mark expected errors in comments for error-triggering tests

# Type Safety with TypeScript

- I MUST write the entire application in TypeScript with strict mode enabled in tsconfig.json.
- I MUST NEVER use `any`; I MUST use `unknown` or generics for complex types instead.
- I MUST create interfaces and types for all data models (e.g., Profile for user data).
- I MUST use explicit type annotations for function parameters and return values.
- I MUST integrate Supabase's type generation using the CLI when needed.
- I MUST parameterize the Supabase client with database types: `createClient<Database>(url, anonKey)`.
- I MUST use explicit typing for React components: `function MyComponent({ user }: { user: Profile })`.
- I MUST use zod schemas for runtime validation and inference with `z.infer<typeof schema>`.
- I SHOULD configure TS lint rules to disallow implicit any and catch type errors early.
- I MUST regenerate types when the database schema changes to catch impacted code.

# Supabase Types and API Definitions

- I MUST ALWAYS use Supabase's generated types for complete type safety when interacting with the database.
- I MUST NEVER directly modify the database.types.ts file because it is autogenerated.
- I MUST parameterize all Supabase client instances with the Database type: `createClient<Database>(url, anonKey)`.
- This ensures that database operations are fully typed:
  - `select()` calls return properly typed data with exact columns and types
  - `insert()` and `update()` operations validate correct fields at compile time
  - `filter()` operations check for valid column names
- When database schema changes occur:
  - I MUST regenerate types immediately with the Supabase CLI
  - I MUST let TypeScript identify all impacted code that needs updating
  - I MUST fix type errors before deploying to prevent runtime issues
- I MUST NEVER use untyped database access; I MUST ALWAYS leverage the generated types to catch errors during development.
- I MUST create type-safe wrapper functions for complex queries that use the generated types.

# Supabase Setup and Security

- Database setup includes customers and subscriptions tables for Paddle integration
- Migration files are stored in supabase/migrations/
- To run migrations: `supabase db push`
- Local development server runs on port 3000 (http://localhost:3000)
- I MUST NEVER modify the database directly. I MUST ALWAYS create migrations.

## Security - Least Privilege Principle

- I MUST use the anon key for public client calls and restrict its rights via RLS policies.
- I MUST NEVER expose the service_role key in client-side code (it bypasses all security).
- I MUST perform privileged operations through server-side functions (Next.js API routes or Edge Functions).
- I MUST validate that the requester is authorized before performing privileged operations.
- I MUST store sensitive config in Next.js environment variables.
- I MUST use the NEXT*PUBLIC* prefix ONLY for values safe to expose (anon key and URL).
- I MUST apply Row Level Security (RLS) policies for ALL tables.
- I SHOULD create secure views or Postgres functions for complex security checks.
- I MUST set appropriate policies for Supabase Storage buckets.
- I MUST NEVER leave any table or storage bucket open by default.

## Storage Configuration

- Storage bucket: 'media' (private bucket)
- Image access requires signed URLs since the bucket is private

## Working with Aggregate Functions

- I MUST use SQL aggregate functions (sum, count, avg) in the database, not client-side calculations
- I MUST use proper column.function() syntax: `column_name.sum()`, `id.count()`, etc.
- I MUST combine multiple aggregations in a single query to reduce database round-trips
- I MUST handle aggregate function results appropriately:

  ```javascript
  // For a query like this:
  .select('id.count(), amount.sum(), rating.avg()')

  // Results will have this structure:
  {
    count: 42,  // NOT id or id_count
    sum: 1500,  // NOT amount or amount_sum
    avg: 4.7    // NOT rating or rating_avg
  }
  ```

- I MUST add explicit type handling for aggregate results since they may be null if no rows match

## Using the mcp_supabase_query Tool

- I MUST use it ONLY for READ operations (SELECT queries) to inspect the database
- Before writing database-related code, I SHOULD run schema inspection queries:
  ```sql
  SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';
  ```
  ```sql
  SELECT column_name, data_type, is_nullable FROM information_schema.columns
  WHERE table_schema = 'public' AND table_name = '[TABLE_NAME]';
  ```
- For testing modifications, I MUST use transactions with ROLLBACK:
  ```sql
  BEGIN;
  -- Your test query here
  ROLLBACK;
  ```
- I MUST NEVER use the tool for INSERT, UPDATE, DELETE, ALTER, CREATE operations
- I MUST ALWAYS limit the number of rows returned: `SELECT * FROM [table_name] LIMIT 10;`
- I MUST use selective columns in joins rather than selecting everything

# Performance Optimization

- I MUST use Server Components by default, only convert to Client Components when necessary.
- I SHOULD implement code-splitting via dynamic imports for heavy components using `next/dynamic`.
- I SHOULD split large third-party libraries and only load them when needed.
- I MUST use Next.js Image component with proper width/height to prevent layout shifts.
- I SHOULD add `priority` to key images (LCP elements) to preload them for faster paint.
- I SHOULD define allowed image domains in next.config.js for security and optimization.
- I MUST use Next.js caching with `cache: 'force-cache'` for static data and `cache: 'no-store'` for dynamic data.
- I SHOULD implement client-side caching with SWR or React Query for Supabase requests.
- I SHOULD set appropriate HTTP cache headers in API responses.
- I SHOULD use Incremental Static Regeneration (ISR) with revalidation for semi-static content.
- I SHOULD apply proper memoization for expensive computations using useMemo and useCallback.
- I MUST keep bundle size small by avoiding large dependencies.
- I SHOULD optimize media uploads by compressing images and using appropriate video formats.
- I SHOULD implement progressive loading for larger media files.
- I SHOULD use Suspense boundaries and skeleton screens for improved loading UX.
- I MUST ALWAYS paginate data that could grow large.
- I MUST minimize client-side JavaScript and leverage server components.
- I SHOULD set performance budgets (e.g., <200KB JavaScript per page, LCP under 2s on fast 3G).
- I SHOULD measure performance using Lighthouse or Web Vitals.
- I SHOULD optimize third-party scripts using Next.js Script component with appropriate strategies.
- I SHOULD use React DevTools profiler to identify and fix unnecessary re-renders.

## Next.js-Specific Best Practices

- I MUST use data fetching methods appropriately:

  - `getStaticProps` for prerendered content
  - ISR with revalidation intervals for semi-static data
  - `getServerSideProps` only when necessary
  - React's streaming support for progressive rendering

- I MUST optimize routing:

  - Use shallow routing for URL changes that don't require data refetching
  - Set priority on preloaded routes with `next/link`
  - Use declarative `<Link>` rather than imperative navigation

- I MUST structure app directory efficiently:

  - Use route segments to minimize layout re-renders
  - Leverage parallel routes for complex layouts with independent data fetching
  - Create focused layouts to avoid unnecessary re-renders
  - Use route interception for modal patterns

- I SHOULD optimize fonts:

  - Use `next/font` to optimize and load custom fonts
  - Apply size-adjust and font-display swap
  - Limit font variants to only those needed

- I MUST NEVER use these anti-patterns:
  - No data fetching duplication between getStaticPaths and getStaticProps
  - Don't mix client and server data fetching for the same data
  - Minimize client-side state when server-rendering is possible
  - Don't nest layouts unnecessarily

# URL-Based State Management

- I MUST store all significant interaction and view state in the URL:

  - Use route parameters for primary resources (e.g., `/dashboard/instagram/:contentId`)
  - Use query parameters for filters, sorting, pagination (e.g., `?filter=recent&sort=date`)
  - Use hash fragments for scroll positions or tabs (e.g., `#comments`)

- I MUST create separate pages with distinct URLs for each major phase of a workflow:

  - For upload/processing/result workflows, I MUST use distinct URLs:
    - Upload form should have its own page (e.g., `/upload`)
    - Results should have their own page with resource ID (e.g., `/results/:resultId`)
    - Never combine upload and result display on the same page
  - For multi-step processes, each logical step MUST have its own URL path
  - All transitions between workflow states MUST be reflected in URL changes
  - Each page MUST have a single, clear responsibility

- I MUST make modals and dialogs URL-addressable:

  - Create dedicated routes for important modals
  - Use URL query parameters to track modal state
  - Handle browser back/forward navigation with modals

- I MUST reflect search functionality in URLs:

  - All search parameters should be in query parameters
  - Preserve filters between page navigations
  - Support deep linking to search results

- I MUST preserve list view states in URLs:

  - Current pagination page
  - Active filters and sorting options
  - Selected view modes and items per page

- I MUST track form workflows in URLs:

  - Use URL parameters for multi-step form progress
  - Preserve form values on page reload
  - Make form errors retrievable after refresh

- I MUST implement bidirectional sync between URL parameters and React state
- I MUST ensure proper URL parameter encoding/decoding for special characters

# Security Best Practices

- I MUST NEVER interpolate untrusted data directly into dangerouslySetInnerHTML.
- I MUST use DOMPurify to sanitize user-generated content when HTML rendering is necessary.
- I MUST use parameterized queries or Supabase RPC calls for any custom SQL.
- I MUST validate inputs on both client and server side.
- I SHOULD implement a strict Content Security Policy in next.config.js.
- I MUST only allow scripts and resources from trusted sources.
- I MUST set X-Frame-Options: DENY unless iframing is explicitly needed.
- I SHOULD monitor CSP violation reports to detect attempted injections.
- I MUST rely on Supabase's secure authentication flows - NEVER create custom ones.
- I MUST set HttpOnly, Secure, and SameSite attributes for cookie-based sessions.
- I SHOULD use Next.js Middleware to protect sensitive routes at the edge.
- I SHOULD implement rate-limiting for authentication-related API routes.

# Error Handling Standards

- I MUST NEVER use silent fallbacks - failing operations MUST produce explicit errors
- I MUST ALWAYS throw or return appropriate error objects with descriptive messages
- I MUST use try/catch blocks sparingly and only where error recovery is actually possible
- I SHOULD log errors with sufficient context for debugging
- I MUST return helpful and meaningful error messages to the user when operations fail
- I MUST implement proper validation before operations that could fail
- I MUST ensure errors are properly propagated in promise chains for async functions
- I MUST use appropriate HTTP status codes for API error responses
- I SHOULD create custom error classes for different error categories when appropriate

# Testing Strategy

- I SHOULD use Vitest/Jest with React Testing Library for component testing
- I MUST focus on testing observable behavior rather than implementation details
- I SHOULD maintain high coverage (90%+) for core business logic
- I SHOULD test integration between components, pages, and Supabase
- I SHOULD use a test Supabase instance or mock responses for testing
- I SHOULD write E2E tests with Playwright/Cypress for critical user journeys
- I SHOULD run E2E tests on production builds with a dedicated test database
- I SHOULD include accessibility testing in test suites
- I SHOULD block merges if tests fail
- I MUST use mocks sparingly and only when necessary

# CI/CD and Tooling

- I SHOULD use ESLint with Next.js config (`next/core-web-vitals`) and TypeScript rules
- I SHOULD include accessibility linting with `eslint-plugin-jsx-a11y`
- I SHOULD use Prettier for code formatting via `eslint-config-prettier`
- I SHOULD set up GitHub Actions to run on every push/PR
- I SHOULD run linting, type-checking, building, and testing in CI
- I SHOULD connect repository to Vercel for automatic deployments
- I SHOULD use Preview Deployments for QA on each PR
- I SHOULD set up Husky for pre-commit and pre-push hooks
- I SHOULD configure error monitoring with Sentry for production

# Figma Design Implementation

## Design Extraction and Accuracy

- I MUST accurately implement all designs from Figma with pixel-perfect precision
- I MUST extract exact color codes from Figma designs and define them as theme variables using the @theme directive in the CSS file:
  ```css
  @import 'tailwindcss';
  @theme {
    --color-brand-primary: #3b82f6;
    --color-brand-secondary: #1e40af;
    /* Add all other colors from Figma */
  }
  ```
- I MUST extract and implement exact spacing, sizing, and layout dimensions from Figma designs
- I MUST match typography styles precisely, including font family, weight, size, line height, and letter spacing
- I MUST translate Figma effects (shadows, blurs, etc.) accurately to their CSS equivalents
- I MUST properly implement all animations and transitions as specified in Figma prototypes
- I MUST maintain design consistency across all breakpoints and responsive variations
- I MUST always download all images from Figma using the mcp_figma_download_figma_images. I will NOT create images myself, I will download and use all of the images from the figma design instead.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samyhajar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
