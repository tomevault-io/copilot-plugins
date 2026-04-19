## replica-nutrient

> You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.


  You are an expert in TypeScript, Node.js, Next.js App Router, React, Shadcn UI, Radix UI and Tailwind.
  
  Code Style and Structure
  - Write concise, technical TypeScript code with accurate examples.
  - Use functional and declarative programming patterns; avoid classes.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
  - Structure files: exported component, subcomponents, helpers, static content, types.
  
  Naming Conventions
  - Use lowercase with dashes for directories (e.g., components/auth-wizard).
  - Favor named exports for components.
  
  TypeScript Usage
  - Use TypeScript for all code; prefer interfaces over types.
  - Avoid enums; use maps instead.
  - Use functional components with TypeScript interfaces.
  
  Syntax and Formatting
  - Use the "function" keyword for pure functions.
  - Avoid unnecessary curly braces in conditionals; use concise syntax for simple statements.
  - Use declarative JSX.
  
  UI and Styling
  - Use Shadcn UI, Radix, and Tailwind for components and styling.
  - Implement responsive design with Tailwind CSS; use a mobile-first approach.
  
  Performance Optimization
  - Minimize 'use client', 'useEffect', and 'setState'; favor React Server Components (RSC).
  - Wrap client components in Suspense with fallback.
  - Use dynamic loading for non-critical components.
  
  Key Conventions
  - Use 'nuqs' for URL search parameter state management.
  - Optimize Web Vitals (LCP, CLS, FID).
  - Limit 'use client':
    - Favor server components and Next.js SSR.
    - Use only for Web API access in small components.
    - Avoid for data fetching or state management.
  
  Follow Next.js docs for Data Fetching, Rendering, and Routing.

  Components
  - Ensure each component has a single responsibility to maintain focus and simplicity.
  - Organize components in a structured way with subfolders for components, hooks, and types.
  - Split large components into smaller, reusable ones to enhance modularity.
  - Use error boundaries for larger components to handle errors gracefully.
  - Define props as types and pass them to components for type safety.
  - Extract logic into custom hooks to keep components clean and focused on presentation.
  - Use useMemo and useCallback to optimize performance by preventing unnecessary computations and re-renders.

  Fetching Data
  - Use React Query for data fetching to manage caching, retries, and background refetching efficiently.
  - Implement proper error handling in components using React Query's error states for user feedback.
  - Create reusable hooks for common data-fetching scenarios to reduce code duplication and ensure consistency.

  File Colocation
  - Place components, functions, and types close to where they are used for better accessibility.
  - Each page should have its own folder with related components and functions for modularity.
  - Place shared components or functions at the nearest common ancestor in the directory hierarchy to avoid complexity.

  Functions
  - Name functions clearly to indicate their purpose for better readability.
  - Ensure functions do only one thing, adhering to the Single Responsibility Principle.
  - Place functions in appropriate locations based on their usage (see File Colocation rules).
  - Implement error handling and logging within functions to improve debugging and monitoring.
  - Use descriptive parameter names and group parameters into objects or types when necessary for clarity.

  Logging
  - Use the logger which is located at src/app/functions/log.ts
  - Utilize the provided logging abstraction (e.g., Sentry wrapper) for consistent logging across the codebase.
  - Log important events such as function starts, errors, and user interactions for traceability.
  - Use log.context to provide additional context for logs to aid in debugging.
  - The logger is imported and used like this:
  import { log } from "@/app/functions/log";
  log.info("Something happened", {
    userId: 123,
    userName: "John Doe",
  });
  log.error("Internal Server Error while switching institution", error);

  Mutating Data
  - Create UI components that trigger mutations and display data for user interaction.
  - Use optimistic updates via hooks for immediate UI feedback before server confirmation.
  - Handle validation and database updates on the server side using server actions for security and consistency.
  Code Example: 
  ```ts
"use server";

import { z } from "zod";
import { prisma } from "@/src/app/misc/singletons/prisma";
import { log } from "@/src/app/functions/log";
import { hasPermission } from "@/src/app/functions/server/roles/permission";

// Define validation schema using Zod
const updateCourseSchema = z.object({
  courseId: z.string().min(1, "Course ID is required"),
  name: z
    .string()
    .min(1, "Name cannot be empty")
    .max(100, "Name must be 100 characters or less"),
});

/**
 * Server action to update a course name.
 * Validates input and updates the database securely.
 * @param {string} courseId - The ID of the course to update.
 * @param {string} name - The new name for the course.
 * @returns {Promise<object>} - The updated course object.
 * @throws {Error} - If validation or update fails.
 */
export async function updateCourseName(courseId: string, name: string) {
  try {
    // Log the action for traceability
    log.info("Attempting to update course name", { courseId, name });

    // Validate input using Zod schema
    const validatedData = updateCourseSchema.parse({ courseId, name });

    // Check user permissions (example utility function)
    await hasPermission("update:course-name", validatedData.courseId);

    // Update the database using Prisma
    const updatedCourse = await prisma.course.update({
      where: { id: validatedData.courseId },
      data: { name: validatedData.name },
    });

    log.info("Course name updated successfully", { courseId });
    return updatedCourse;
  } catch (error) {
    // Log the error for debugging
    log.error("Failed to update course name", error);
    throw new Error("Failed to update course");
  }
}
  ``` 

  Naming Conventions
  - Use kebab-case for file names (e.g., my-component.tsx) to maintain consistency.
  - Use snake_case for translation keys (e.g., my_translation_key) to ensure runtime validation.
  - Use camelCase for everything else, including function names (e.g., myFunction), component names (e.g., MyComponent), variable names (e.g., myVariable), and other identifiers, to follow JavaScript and React conventions.

  Colocate types with relevant files or move shared types to a global src/app/types/ directory for organization.

  Performance
  - Use dynamic imports (e.g., next/dynamic) to split code and reduce initial load times.
  - Use useMemo, React.memo, and useCallback to prevent unnecessary re-renders and computations for better performance.

  Server-Only and Client-Only Code
  - Use server-only and client-only packages to separate server-side and client-side code, ensuring proper isolation.

  State Management
  - Use useQueryState for managing URL and query-based state (e.g., filters, pagination).
  - Use Context + Zustand for sharing state within a specific section of the app (e.g., a form or dashboard).
  - Use Global Zustand for application-wide state accessible across multiple areas (e.g., user authentication).

  TypeScript
  - Always define types for variables, parameters, and return values to ensure type safety.
  - Explicitly declare types to avoid implicit any, preferring unknown for edge cases.
  - Group related types together and colocate them with relevant modules or move to a global directory when shared.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AAlber) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
