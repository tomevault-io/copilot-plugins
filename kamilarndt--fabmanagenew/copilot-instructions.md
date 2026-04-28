## fabmanagenew

> TypeScript excellence rules for FabManage-Clean


# TypeScript Rules for FabManage-Clean

- Enable strict mode; never use any or unknown.
- Prefer interface for component props and public shapes; use type for data and unions.
- Export named components/functions; avoid default exports.
- Explicit return types for exported functions and custom hooks.
- Avoid type assertions; use type guards and Zod schemas.
- Keep side effects out of modules; prefer pure functions.
- Model business entities with rich types (modules, tiles, CNC, materials).

## Component Props

```tsx
// ✅ Good - Interface for component props
interface ButtonProps {
  variant?:
    | "default"
    | "destructive"
    | "outline"
    | "secondary"
    | "ghost"
    | "link";
  size?: "default" | "sm" | "lg" | "icon";
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  className?: string;
}

// ✅ Good - Generic component props
interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  onRowClick?: (row: T) => void;
  loading?: boolean;
}
```

## Business Entities

```tsx
// ✅ Good - Rich business types
interface ProjectModule {
  id: string;
  name:
    | "pricing"
    | "concept"
    | "technical_design"
    | "production"
    | "materials"
    | "logistics";
  isEnabled: boolean;
  completedAt?: string; // ISO
  dependencies?: string[];
  estimatedDuration?: number; // hours
}

export interface Project {
  id: string;
  name: string;
  modules: ProjectModule[];
  status: "draft" | "active" | "completed" | "archived";
  createdAt: string;
  updatedAt: string;
  clientId?: string;
  priority: "low" | "medium" | "high" | "urgent";
}
```

## Design Tokens

```tsx
// ✅ Good - Typed design tokens
export type ColorToken = keyof typeof designTokens.colors;
export type SpacingToken = keyof typeof designTokens.spacing;
export type RadiusToken = keyof typeof designTokens.radius;

interface ThemeProps {
  primaryColor: ColorToken;
  spacing: SpacingToken;
  borderRadius: RadiusToken;
}
```

## API Types

```tsx
// ✅ Good - API response types
interface ApiResponse<T> {
  data: T;
  error: string | null;
  loading: boolean;
}

interface CreateProjectRequest {
  name: string;
  clientId: string;
  priority: Project["priority"];
  modules: Omit<ProjectModule, "id" | "completedAt">[];
}

type CreateProjectResponse = ApiResponse<Project>;
```

## Utility Types

```tsx
// ✅ Good - Utility types for forms
type FormData<T> = {
  [K in keyof T]: T[K] extends string ? string : T[K];
};

type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// Usage
type CreateProjectForm = FormData<CreateProjectRequest>;
type UpdateProject = PartialBy<Project, "id" | "createdAt">;
```

## Error Handling

```tsx
// ✅ Good - Typed error handling
interface AppError {
  code: string;
  message: string;
  details?: Record<string, unknown>;
}

type Result<T, E = AppError> =
  | { success: true; data: T }
  | { success: false; error: E };

// Usage
async function fetchProject(id: string): Promise<Result<Project>> {
  try {
    const project = await projectService.get(id);
    return { success: true, data: project };
  } catch (error) {
    return {
      success: false,
      error: { code: "FETCH_ERROR", message: "Failed to fetch project" },
    };
  }
}
```

## Hook Types

```tsx
// ✅ Good - Custom hook return types
interface UseProjectReturn {
  project: Project | null;
  loading: boolean;
  error: string | null;
  updateProject: (updates: Partial<Project>) => Promise<void>;
  deleteProject: () => Promise<void>;
}

export function useProject(id: string): UseProjectReturn {
  // implementation
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamilarndt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
