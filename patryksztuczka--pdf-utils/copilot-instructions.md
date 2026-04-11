## pdf-utils

> This document provides comprehensive instructions and conventions for AI agents operating within the `pdf-util` repository. Adhering to these guidelines ensures consistency, maintainability, and compatibility with the project's architecture.

# Agent Guidelines for pdf-util

This document provides comprehensive instructions and conventions for AI agents operating within the `pdf-util` repository. Adhering to these guidelines ensures consistency, maintainability, and compatibility with the project's architecture.

## 1. Environment & Commands

### Package Management
- Use **Bun** (`bun`) for all package management tasks (installing, adding, removing dependencies).
- The `bun.lock` file is the source of truth for dependency versions.

### Development Workflow
- **Start Development Server:** `bun run dev`
- **Build for Production:** `bun run build` (executes `tsc -b` followed by `vite build`).
- **Preview Production Build:** `bun run preview`

### Quality Control
- **Linting:** `bun run lint` (runs ESLint). Fix all linting errors before committing.
- **Type Checking:** `bun run typecheck` (runs `tsc -b`). This is critical for catching TypeScript errors that ESLint might miss.

### Testing
- **Current Status:** No testing framework is currently configured.
- **Action:** If you need to add tests, install **Vitest**.
- **Running Tests (Recommended Config):**
  - All tests: `bunx vitest`
  - Single test file: `bunx vitest src/path/to/file.test.ts`
  - Single test by name: `bunx vitest -t "test name"`
  - Watch mode: `bunx vitest --watch`

## 2. Code Style & Architecture

### Directory Structure
- `src/components/`: Modular React components.
  - `landing/`: Landing page specific components.
  - `merge/`, `split/`, `organizer/`: Feature-specific tool components.
  - `shared/`: Reusable components used across multiple features (e.g., `PDFThumbnail`).
  - `ui/`: Base UI primitives based on **shadcn/ui** patterns.
- `src/hooks/`: Custom React hooks (e.g., `use-pdf.ts`, `use-theme.ts`).
- `src/lib/`: Utility functions and library wrappers (e.g., `pdf-utils.ts`, `pdf-render.ts`).
- `src/providers/`: React Context providers and their initial state/logic.
- `src/data-access-layer/`: (Proposed) Logic for external API interactions.
- `src/schemas/`: (Proposed) Zod schemas for data validation.

### Components
- **Syntax:** Always use the `function` keyword for components.
  ```tsx
  export function MyComponent({ prop1 }: MyComponentProps) {
    return <div>{prop1}</div>;
  }
  ```
- **Exports:** Use **named exports** only. No `default` exports for components.
- **Props:** Define props using an `interface` named `[ComponentName]Props`.
- **Logic Placement:** Keep hooks and state at the top level of the component.
- **Visuals:** Prefer **Lucide React** for icons and **shadcn/ui** for UI elements.

### Styling with Tailwind CSS v4
- **Framework:** **Tailwind CSS v4** is used.
- **Class Merging:** ALWAYS use the `cn(...)` utility from `@/lib/utils` for conditional classes.
  ```tsx
  import { cn } from "@/lib/utils";
  
  export function Component({ className, isActive }: Props) {
    return <div className={cn("base-classes", isActive && "active-classes", className)} />;
  }
  ```
- **Conventions:** Standard Tailwind kebab-case for utility classes. Avoid custom CSS unless absolutely necessary.

### TypeScript & Types
- **Strict Mode:** TypeScript is in strict mode. Do not use `any` unless there is an extremely compelling reason.
- **Imports:**
  - Use absolute imports with the `@/` alias (resolves to `src/`).
  - Use type-only imports for types: `import type { PDFDocumentProxy } from "pdfjs-dist";`.
- **Definitions:**
  - `interface` for object shapes and component props.
  - `type` for unions, intersections, and aliases.
- **Verbatim Module Syntax:** Adhere to `verbatimModuleSyntax: true` from `tsconfig.json`.

### Naming Conventions
- **Files:** `kebab-case.tsx` or `kebab-case.ts`.
- **Folders:** `kebab-case` for all directories.
- **Components:** `PascalCase`.
- **Hooks:** `use-camelCase` (e.g., `use-pdfDocument.ts`).
- **Functions/Variables:** `camelCase`.
- **Constants:** `UPPER_SNAKE_CASE` for global constants.

## 3. Implementation Details

### PDF Processing
- **Libraries:** Uses `pdf-lib` for document manipulation (merging, splitting) and `pdfjs-dist` for rendering previews.
- **Client-Side:** All processing MUST remain client-side. Do not introduce server-side dependencies for PDF handling.
- **Rendering:** Use the `canvas` based rendering pattern established in `src/lib/pdf-render.ts`.

### State Management
- **Local State:** Use `useState` and `useReducer` for component-level state.
- **Global State:** Use React Context (established in `src/providers/`).
- **Server State:** If introducing server interactions, use **TanStack React Query**.

### UI & Accessibility
- **Icons:** Use `lucide-react`. Ensure icons have appropriate sizes (usually `h-4 w-4` or `h-5 w-5`).
- **Buttons:** Use the `Button` component from `@/components/ui/button`.
- **Feedback:** Use `Toast` (if available) or inline alerts for user feedback on actions.
- **Responsiveness:** Use Tailwind's responsive prefixes (`md:`, `lg:`, etc.) to ensure the tools work on various screen sizes.

### Error Handling
- **Graceful Failures:** Use `try...catch` blocks for async operations, especially file reading and PDF manipulation.
- **User Feedback:** Provide clear error messages in the UI, especially for file processing failures.
- **Loading States:** Always handle and display loading states during PDF processing or rendering.

### Performance
- **Large Files:** Be mindful of memory usage when handling large PDFs. Avoid holding large `Uint8Array`s in state if not necessary.
- **Cleanup:** Ensure `pdf.destroy()` is called in `useEffect` cleanups when using `pdfjs-dist` proxies to prevent memory leaks.

## 4. Git Conventions

### Commit Messages
- Use **Conventional Commits** (e.g., `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`).
- Keep descriptions concise and in the imperative mood (e.g., `feat: add merge tool functionality`).

### Workflow
- Ensure `bun run lint` and `bun run typecheck` pass before final delivery.
- Create modular commits that focus on a single logical change.

---
_Last updated: January 2026_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patryksztuczka)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/patryksztuczka)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
