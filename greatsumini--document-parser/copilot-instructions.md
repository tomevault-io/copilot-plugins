## vooster-guideline

> This project is a cloud-native SaaS web application that enables users to upload various data files (Excel, CSV, PDF, TXT) and automatically generate hospital-compliant Markdown documents using LLM (OpenAI GPT-4o). The stack comprises Next.js 15 (App Router, server components), TypeScript, tRPC, Supabase (PostgreSQL & Storage), OpenAI SDK, Vercel AI SDK, Shadcn UI, Tailwind CSS, and modern DevOps (Vercel, GitHub Actions, Sentry).

# Project Code Guideline

---

## 1. Project Overview

This project is a cloud-native SaaS web application that enables users to upload various data files (Excel, CSV, PDF, TXT) and automatically generate hospital-compliant Markdown documents using LLM (OpenAI GPT-4o). The stack comprises Next.js 15 (App Router, server components), TypeScript, tRPC, Supabase (PostgreSQL & Storage), OpenAI SDK, Vercel AI SDK, Shadcn UI, Tailwind CSS, and modern DevOps (Vercel, GitHub Actions, Sentry).  
**Key architectural decisions:**
- Monorepo structure with domain-driven organization.
- Serverless-first deployment (Vercel Edge/Serverless, Supabase).
- Strong boundaries between presentation, business logic, and infrastructure.
- Type safety and input validation across the stack (TypeScript, Zod).
- Real-time, streaming, and batch data flows.

---

## 2. Core Principles

1. **Type Safety First:** All code MUST use TypeScript with strict type checks enabled.
2. **Single Responsibility:** Each module/component MUST have only one clear responsibility.
3. **Explicit Error Handling:** All external calls and user input MUST be validated and errors handled gracefully.
4. **Domain-Centric Organization:** Code MUST be organized by business domain, not by technical layer.
5. **Security by Default:** Sensitive operations and data access MUST follow least privilege and encryption requirements.

---

## 3. Language-Specific Guidelines

### 3.1 TypeScript & Next.js

#### File Organization and Directory Structure

- **MUST:** Follow the monorepo and domain-driven folder structure as specified:

    ```
    /apps/web/
      app/                  // Next.js app router structure
      components/           // Reusable UI components
      features/{domain}/    // Domain features (e.g., report, template, auth)
      lib/                  // Utilities and API clients
      styles/               // Tailwind and global styles
    /packages/
      api/                  // tRPC routers and procedures
      db/                   // Prisma schema and DB utilities
      llm/                  // LLM prompt helpers and adapters
      shared/types/         // Shared TypeScript types
      shared/utils/         // Shared utility functions
      shared/constants/     // Shared constants
    /infra/                 // Deployment, scripts, config
    ```

- **MUST:** Place each feature in its own folder under `/features/{domain}` with clear separation of UI, hooks, and logic.

```typescript
// MUST: Example feature folder structure
/features/report/
  ReportEditor.tsx
  useReportData.ts
  reportUtils.ts
```

- **MUST NOT:** Place unrelated components, utilities, or logic in a single file or folder.

#### Import/Dependency Management

- **MUST:** Use absolute imports from the project root, not relative paths traversing multiple directories.

```typescript
// MUST: Use absolute imports for clarity and maintainability
import { parseExcel } from 'features/report/reportUtils'
```

- **MUST:** Group external imports before internal imports, and order alphabetically within each group.

- **MUST:** Keep dependencies minimal and only import what is required for the module’s responsibility.

- **MUST NOT:** Use wildcard imports (`import * as ...`) except for TypeScript enums or namespaces.

#### Error Handling Patterns

- **MUST:** Validate all user inputs and API payloads using Zod schemas, both client- and server-side.

```typescript
// MUST: Zod validation for incoming API data
import { z } from 'zod'

const UploadSchema = z.object({
  file: z.instanceof(File),
  templateId: z.string().uuid(),
})

export const uploadHandler = (data: unknown) => {
  const parsed = UploadSchema.safeParse(data)
  if (!parsed.success) {
    throw new Error('Invalid input data')
  }
  // Proceed with validated data
}
```

- **MUST:** Handle all async errors using try/catch and surface user-friendly messages.

```typescript
// MUST: Graceful async error handling
try {
  const result = await api.createDocument(payload)
} catch (error) {
  logger.error(error)
  showToast('Failed to create document. Please try again.')
}
```

- **MUST NOT:** Swallow errors or use empty catch blocks.

---

### 3.2 tRPC & API Layer

- **MUST:** Separate routers by domain (`reportRouter`, `templateRouter`, `authRouter`).
- **MUST:** Define input/output types using Zod and TypeScript generics.
- **MUST:** Use context for authentication and authorization checks on each procedure.
- **MUST NOT:** Mix unrelated procedures in the same router or expose raw database models directly.

---

### 3.3 Prisma & Supabase Integration

- **MUST:** Use Prisma Client for all database access.
- **MUST:** Use transactions for multi-step DB operations (e.g., document + version record).
- **MUST:** Never expose raw SQL queries in business logic.
- **MUST:** Use Row Level Security (RLS) for all data access.
- **MUST NOT:** Hardcode credentials or secrets in source files.

---

### 3.4 React (Client Components)

- **MUST:** Use functional components with explicit typing (`FC<Props>` or `function Component(props: Props)`).
- **MUST:** Use Zustand for local UI state, react-query for server state.
- **MUST:** Co-locate component, hook, and style files within the feature directory.
- **MUST NOT:** Use class components or mix stateful logic in presentational components.

---

### 3.5 Styling (Tailwind CSS, Shadcn UI)

- **MUST:** Use Tailwind utility classes for layout and spacing.
- **MUST:** Use Shadcn UI components for all standard UI elements.
- **MUST:** Extract repetitive style logic into reusable Tailwind classnames or component props.
- **MUST NOT:** Write custom CSS unless required for unique features.

---

## 4. Code Style Rules

### 4.1 MUST Follow

- **Consistent Naming:** Use `camelCase` for variables/functions, `PascalCase` for components/types, `UPPER_SNAKE_CASE` for constants.
    - *Rationale:* Ensures readability and reduces naming conflicts.

- **Strict Linting:** All code MUST pass ESLint (`@typescript-eslint`), Prettier, and project-specific lint rules before merge.
    - *Rationale:* Prevents style drift and enforces consistency.

- **Type Annotations:** All function parameters and return values MUST be explicitly typed.
    - *Rationale:* Maximizes type safety and self-documentation.

- **Single Export per File:** Each file MUST export only one main component, hook, or utility.
    - *Rationale:* Simplifies imports and encourages single responsibility.

- **Test Coverage:** All business logic and API handlers MUST have unit or integration tests.
    - *Rationale:* Ensures reliability and enables safe refactoring.

```typescript
// MUST: Explicit typing and single responsibility
export function mapExcelToFields(data: ExcelData): MappedFields {
  // ...
}
```

### 4.2 MUST NOT Do

- **No God Files:** MUST NOT create files with multiple unrelated components, hooks, or utilities.
    - *Reason:* Violates separation of concerns, hard to maintain.

```typescript
// MUST NOT: Multiple unrelated exports in one file
export function parseExcel() { /* ... */ }
export function sendSlackAlert() { /* ... */ } // unrelated
```

- **No Complex State in Single Component:** MUST NOT handle global or cross-domain state within a single React component.
    - *Reason:* Leads to untestable, tightly coupled code.

- **No Direct DB/LLM Calls in UI:** UI components MUST NOT directly access database or external APIs.
    - *Reason:* Breaks abstraction, hinders testability, and violates layering.

- **No Magic Strings/Numbers:** MUST NOT use hardcoded values for keys, roles, or config; use constants/enums.
    - *Reason:* Reduces errors and simplifies updates.

- **No Silent Failures:** MUST NOT catch errors without logging or user notification.

---

## 5. Architecture Patterns

### 5.1 Component/Module Structure

- **MUST:** Structure modules by domain, with clear separation between presentation, hooks, logic, and API.
- **MUST:** Export only the main entry point from each feature directory.

```typescript
// MUST: Feature module structure
/features/template/
  TemplateList.tsx
  useTemplates.ts
  templateApi.ts
```

### 5.2 Data Flow Patterns

- **MUST:** Use tRPC for all client-server communication except for file uploads (use REST/multipart).
- **MUST:** Use react-query for all server-side state fetching/mutations.
- **MUST:** Use SSE/streaming for LLM document generation updates.

```typescript
// MUST: tRPC mutation for creating document
const mutation = trpc.report.create.useMutation({
  onSuccess: () => refetchReports(),
})
```

### 5.3 State Management Conventions

- **MUST:** Use Zustand for local UI state (e.g., modals, theme).
- **MUST:** Use react-query for all server state and data synchronization.
- **MUST NOT:** Use Redux, MobX, or custom global state libraries.

### 5.4 API Design Standards

- **MUST:** All tRPC procedures MUST have Zod-validated input/output and enforce authentication via context.
- **MUST:** REST endpoints (e.g., file upload) MUST validate file type, size, and permissions server-side.
- **MUST:** Return user-friendly, actionable error messages.

```typescript
// MUST: tRPC procedure with Zod validation and context
import { z } from 'zod'
import { protectedProcedure } from '../trpc'

export const createTemplate = protectedProcedure
  .input(z.object({ name: z.string(), fields: z.array(z.string()) }))
  .mutation(async ({ input, ctx }) => {
    // Business logic here
  })
```

---

## Example Code Snippets

```typescript
// MUST: Proper domain separation and explicit typing
// features/report/ReportEditor.tsx
import { useReportData } from './useReportData'

export const ReportEditor: React.FC = () => {
  const { data, isLoading } = useReportData()
  // ...
}
```

```typescript
// MUST NOT: Mixing unrelated logic in one file
// This file contains both report and template logic, which is incorrect.
export const ReportEditor = () => { /* ... */ }
export const TemplateManager = () => { /* ... */ }
```

```typescript
// MUST: Zod validation for API input
const InputSchema = z.object({
  templateId: z.string().uuid(),
  file: z.instanceof(File),
})
```

```typescript
// MUST: react-query for server state
const { data, refetch } = useQuery(['templates'], fetchTemplates)
```

```typescript
// MUST: Zustand for UI state
import create from 'zustand'

type UIState = {
  isModalOpen: boolean
  openModal: () => void
  closeModal: () => void
}

export const useUIStore = create<UIState>((set) => ({
  isModalOpen: false,
  openModal: () => set({ isModalOpen: true }),
  closeModal: () => set({ isModalOpen: false }),
}))
```

---

## Quality Criteria

- All code MUST be type-safe, domain-oriented, and strictly separated by responsibility.
- All APIs MUST enforce validation, authentication, and error handling.
- All state management MUST use the prescribed libraries and patterns.
- All code MUST be reviewed, linted, and tested before merge.
- All sensitive data MUST be encrypted and accessed via secure, audited methods.

---

This code guideline is the single source of truth for all contributors. Adherence is mandatory for all code submissions.

---
> Source: [greatSumini/document-parser](https://github.com/greatSumini/document-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
