## snapconnect

> This document outlines the best practices and conventions for the SnapConnect

# Gemini Code Assist Rules

This document outlines the best practices and conventions for the SnapConnect
project. By following these guidelines, we ensure the codebase remains clean,
consistent, and maintainable.

## Core Technologies

- **React Native**: For cross-platform mobile app development.
- **TypeScript**: For static typing and improved code quality.
- **Supabase**: As the backend for authentication, database, and storage.
- **Expo**: To streamline development, building, and deployment.

---

## 1. General Coding Best Practices

These are the foundational principles for all code in the project.

- **Clean Code**: Write self-documenting code with meaningful names (e.g.,
  `fetchUserData` instead of `getStuff`).
- **DRY (Don't Repeat Yourself)**: Avoid duplication by extracting reusable
  logic into hooks, utilities, or services.
- **KISS (Keep It Simple, Stupid)**: Keep solutions simple and focused; avoid
  over-engineering.
- **Single Responsibility Principle**: Functions, components, and modules should
  do one thing well.
- **Consistency**: Adhere to established naming conventions (`camelCase` for
  variables/functions, `PascalCase` for components/types/interfaces).

### Project Structure

- **`src/components/`**: Reusable UI components.
- **`src/screens/`**: Top-level components for each screen.
- **`src/hooks/`**: Custom React hooks for shared logic.
- **`src/services/`**: Backend logic (Supabase, APIs).
- **`src/utils/`**: Helper functions. Pure, functional-style utilities.
- **`src/types/`**: Shared TypeScript types and interfaces.
- **`src/styles/`**: Global styles and theme definitions.
- **`src/navigation/`**: Navigation stacks and routing logic.
- **`src/stores/`**: State management stores (e.g., Zustand).
- **`supabase/`**: Supabase-specific configurations, migrations, and functions.

Use barrel files (`index.ts`) to simplify imports from directories.

### Error Handling & Logging

- Wrap all asynchronous operations (especially service calls) in `try-catch`
  blocks.
- Use the dedicated logger at `src/utils/logger.ts` for all service calls and
  important events to facilitate debugging.
- Implement error boundaries in React components to prevent UI crashes.

---

## 2. TypeScript Best Practices

- **Strict Mode**: Enable strict null checks and other strict options in
  `tsconfig.json`.
- **Avoid `any`**: Use `unknown` for values with an unknown type and perform
  type-checking.
- **Interfaces over Types**: Prefer `interface` for defining object shapes and
  `type` for union or intersection types.
- **Explicit Types**: Provide explicit types for function arguments and return
  values.
- **Type Guards**: Use type guards to narrow types within conditional blocks.
- **Shared Types**: Define and export shared types from the `src/types/`
  directory. For Supabase, use generated types for database operations.

---

## 3. React & React Native Component Best Practices

- **Functional Components**: Use functional components with hooks.
- **Component Granularity**: Keep components small and focused. Extract complex
  logic into custom hooks.
- **Props**:
  - Use TypeScript interfaces to define props for each component.
  - Destructure props for readability.
  - Provide default values for optional props using default parameter syntax.
- **Styling**:
  - Separate styles into a `styles.ts` file for each component.
  - Use `StyleSheet.create` for performance.
  - Avoid inline styles.
- **Performance**:
  - Use `React.memo` for components that render frequently with the same props.
  - Use `useCallback` to memoize functions passed as props to child components.
  - Use `FlatList` or `SectionList` for long lists of data.
- **Accessibility**: Include `accessibilityLabel` and `accessibilityRole` for
  all interactive elements.

---

## 4. Styling Conventions

- **Global Theme**: Use the global theme defined in `src/styles/theme.ts` for
  consistent colors, fonts, and spacing.
- **Responsive Design**: Use Flexbox for layouts. Use `useWindowDimensions` for
  styles that need to adapt to screen size.
- **Dark Mode**: Support dark mode by using the `useColorScheme` hook and
  defining colors for both light and dark themes.

---

## 5. Supabase & Backend Services

- **Service Layer**: All Supabase logic must be encapsulated within the
  `src/services/` directory. Components should not call Supabase directly.
- **Type Safety**: Use the Supabase client typed with your database schema.
  Define and use `Row`, `Insert`, and `Update` types for tables.
- **Security**:
  - Rely on **Row Level Security (RLS)** for all data access control.
  - Never disable RLS. Policies should be the single source of truth for
    authorization.
  - Use `auth.uid()` in RLS policies to scope data access to the authenticated
    user.
- **Database Functions (RPC)**: Use PostgreSQL functions for complex queries,
  transactions, or business logic that should run on the server.
- **Real-time**: Use Supabase Realtime for live features. Encapsulate channel
  subscriptions within hooks or components and ensure they are cleaned up on
  unmount.
- **Storage**: Organize file uploads into logical buckets. Use RLS policies to
  secure access to storage buckets.
- **Logging**: Log all service function calls, including parameters and outcomes
  (success or error), using the logger utility.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BrooksPoltl)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BrooksPoltl)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
