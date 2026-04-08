## eportal

> This document outlines the architecture, coding conventions, and styling guidelines for the Eportal project. It serves as a reference for the Gemini agent to ensure that all generated code is consistent with the existing codebase.

# Project Gemini Configuration

This document outlines the architecture, coding conventions, and styling guidelines for the Eportal project. It serves as a reference for the Gemini agent to ensure that all generated code is consistent with the existing codebase.

---

## 1. Tech Stack Summary

The project is a full-stack TypeScript monorepo built with Bun and Turborepo.

| Category      | Technology                                       | Notes                                                 |
| ------------- | ------------------------------------------------ | ----------------------------------------------------- |
| **Workspace** | Bun, Turborepo                                   | High-performance TypeScript/JavaScript toolkit.       |
| **Language**  | TypeScript                                       | For end-to-end type safety.                         |
| **Backend**   | [oRPC](https://orpc.dev/)                        | End-to-end typesafe API layer (similar to tRPC).    |
| **Database**  | PostgreSQL + [Drizzle ORM](https://orm.drizzle.team/) | SQL-like ORM with `drizzle-kit` for migrations.     |
| **Frontend**  | React (with Vite)                                | Modern, fast web application framework.             |
| **Routing**   | [TanStack Router](https://tanstack.com/router/)  | Type-safe, file-based routing.                      |
| **Data/State**| [TanStack Query](https://tanstack.com/query/)    | Server-state management, caching, and data fetching.|
| **UI Kit**    | [ShadCN UI](https://ui.shadcn.com/)              | Composable components built on Radix UI.              |
| **Styling**   | [Tailwind CSS](https://tailwindcss.com/)         | Utility-first CSS framework.                        |
| **Forms**     | [React Hook Form](https://react-hook-form.com/)  | Performant form state management.                   |
| **Validation**| [Zod](https://zod.dev/)                          | Schema declaration and validation (used with forms and API). |
| **Tables**    | [TanStack Table](https://tanstack.com/table/)    | Headless UI for building powerful data grids.       |
| **Auth**      | `better-auth`                                    | Project-specific authentication library.              |
| **Icons**     | [Lucide React](https://lucide.dev/)              | Icon library.                                         |


---

## 2. Architecture & Code Patterns

### Monorepo Structure

The project uses a standard Turborepo structure:
- `apps/web`: The main Vite-based React web application.
- `packages/api`: The oRPC backend, defining all API endpoints and procedures.
- `packages/db`: Drizzle ORM schema, configuration, and migration scripts.
- `packages/auth`: Authentication logic, shared between the frontend and backend.

### Backend (oRPC)

The API in `packages/api` exposes type-safe procedures that can be directly called from the frontend client (`apps/web/src/utils/orpc.ts`). This eliminates the need for manual API client fetching code and guarantees that frontend and backend types are always in sync.

### Frontend Patterns

- **Routing**: Routes are defined in the `apps/web/src/routes` directory using a file-based convention. User roles have dedicated sub-directories (e.g., `/student`, `/lecturer`).
- **Component Organization**:
    - **UI Primitives**: Located in `apps/web/src/components/ui`, these are the base ShadCN components.
    - **Domain-Specific Forms**: Reusable form components for specific data models (e.g., creating a `Course` or `Department`) are located in `apps/web/src/components/forms`. They use `react-hook-form` and Zod schemas for validation.
    - **Domain-Specific Tables**: Table components for displaying data (e.g., `UsersTable`, `CoursesTable`) are located in `apps/web/src/components/tables` and are built using `TanStack Table`.
- **State Management**:
    - **Server State**: Managed exclusively by TanStack Query. Use `orpc.query()` for data fetching.
    - **Client State**: For simple component-level state, use `React.useState`. For more complex global state, a simple context or a lightweight store like Zustand would be appropriate if needed.

---

## 3. Styling & UI (ShadCN Theme)

The UI is built with Tailwind CSS and ShadCN UI components. Adherence to the established theme is critical.

### Key Theme Properties

- **Source File**: `apps/web/src/index.css`
- **Color Space**: `oklch`
- **Border Radius**: `0rem` (square corners). The `--radius` variable is `0rem`.
- **Fonts**:
    - **Sans-serif**: `Oxanium`
    - **Serif**: `Merriweather`
    - **Monospace**: `Fira Code`

### Color Palette

Here is a summary of the primary theme colors defined in `index.css`. Always use these variables for consistency.

| Name                 | Light Mode (`:root`)               | Dark Mode (`.dark`)                |
| -------------------- | ---------------------------------- | ---------------------------------- |
| `--background`       | `oklch(0.9885 0.0057 84.5659)`     | `oklch(0.2161 0.0061 56.0434)`     |
| `--foreground`       | `oklch(0.3660 0.0251 49.6085)`     | `oklch(0.9699 0.0013 106.4238)`    |
| `--primary`          | `oklch(0.5553 0.1455 48.9975)`     | `oklch(0.7049 0.1867 47.6044)`     |
| `--primary-foreground`| `oklch(1.0000 0 0)`                | `oklch(1.0000 0 0)`                |
| `--secondary`        | `oklch(0.8276 0.0752 74.4400)`     | `oklch(0.4444 0.0096 73.6390)`     |
| `--card`             | `oklch(0.9686 0.0091 78.2818)`     | `oklch(0.2685 0.0063 34.2976)`     |
| `--border`           | `oklch(0.8866 0.0404 89.6994)`     | `oklch(0.3741 0.0087 67.5582)`     |
| `--destructive`      | `oklch(0.4437 0.1613 26.8994)`     | `oklch(0.5771 0.2152 27.3250)`     |

---

## 4. Naming & File Conventions

- **Directories**: `kebab-case` (e.g., `user-permissions`).
- **Files (Components, Utils)**: `kebab-case.tsx` or `kebab-case.ts` (e.g., `sign-in-form.tsx`).
- **React Components**: `PascalCase` (e.g., `SignInForm`).
- **Database Schema**: `camelCase` for table and column names in Drizzle schema definitions.

---

## 5. Form Handling Conventions

All forms in the application must adhere to the following conventions for consistency, maintainability, and proper data submission.

### 5.1 Form Library & Validation

-   **Form Library**: `@tanstack/react-form` is the standard for building forms.
-   **Validation**: `zod` schemas define form validation rules. The `validatorAdapter` for `zodValidator` is generally **not needed** with the current versions of `@tanstack/react-form` and `@tanstack/zod-form-adapter`.

### 5.2 Form Submission with oRPC and TanStack Query

-   **API Interaction**: Always use `useMutation` from `@tanstack/react-query` to interact with oRPC procedures (`orpc.someResource.create/update`).
-   **Mutation Pattern**:
    -   Define `create` and `update` mutations using `orpc.someResource.create.mutationOptions()` and `orpc.someResource.update.mutationOptions()`.
    -   Include `onSuccess` and `onError` callbacks within `mutationOptions` to handle side effects:
        -   Display `toast` notifications for success or failure.
        -   `queryClient.invalidateQueries` should be called to refresh relevant data in tables/lists.
        -   `form.reset()` should be called on successful creation.
    -   **Triggering Mutations**: Use `mutation.mutateAsync(value)` within the form's `onSubmit` handler.

    ```tsx
    const createMutation = useMutation(
      orpc.faculties.create.mutationOptions({
        onSuccess: (data) => {
          toast.success("Faculty created successfully", { description: \`\${data.name} has been added.\` });
          queryClient.invalidateQueries({ queryKey: orpc.faculties.getAll.queryKey() });
          form.reset();
        },
        onError: (err) => {
          toast.error("Failed to create faculty", { description: err.message });
        },
      })
    );

    const updateMutation = useMutation(
      orpc.faculties.update.mutationOptions({
        onSuccess: (data) => {
          toast.success("Faculty updated successfully", { description: \`\${data.name} has been updated.\` });
          queryClient.invalidateQueries({ queryKey: orpc.faculties.getAll.queryKey() });
        },
        onError: (err) => {
          toast.error("Failed to update faculty", { description: err.message });
        },
      })
    );

    const form = useForm({
        // ... defaultValues
        onSubmit: async ({ value }) => {
            if (mode === "create") {
                await createMutation.mutateAsync(value);
            } else {
                await updateMutation.mutateAsync({ id: someId, ...value });
            }
        }
    })
    ```

### 5.3 Field Structure & Error Display

-   Each form field should use the `form.Field` component from `@tanstack/react-form`.
-   Wrap the `Label` and `Input`/`Select`/`Textarea` within a `div` for consistent spacing (`space-y-1` or `space-y-2`).
-   Display validation errors below the input using `field.state.meta.errors`.

```tsx
<form.Field
  name="fieldName"
  validators={{ onChange: someSchema.shape.fieldName }}
  children={(field) => (
    <div className="space-y-1">
      <Label htmlFor={field.name}>Field Label *</Label>
      <Input
        id={field.name}
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
      {field.state.meta.errors && (
        <p className="text-sm text-red-500">
          {field.state.meta.errors.join(", ")}
        </p>
      )}
    </div>
  )}
/>
```

### 5.4 Dialogs for Forms

-   When a form is presented within a ShadCN `Dialog`, the `DialogContent` **must** include a `DialogHeader` with a `DialogTitle` and `DialogDescription`. This provides context and improves accessibility.
-   Example structure in the parent component:
    ```tsx
    <Dialog>
      <DialogTrigger asChild>
        <Button>Open Form</Button>
      </DialogTrigger>
      <DialogContent className="max-w-2xl">
        <DialogHeader>
          <DialogTitle>Form Title</DialogTitle>
          <DialogDescription>Short description of the form's purpose.</DialogDescription>
        </DialogHeader>
        <MyForm />
      </DialogContent>
    </Dialog>
    ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ayubaezekiel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ayubaezekiel)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
