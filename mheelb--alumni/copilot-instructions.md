## alumni

> A comprehensive platform to manage and track alumni data, featuring a modern web interface, a robust API, and automated scraping capabilities to keep information synchronized.

# Project Context: Alumni Platform

## Goal
A comprehensive platform to manage and track alumni data, featuring a modern web interface, a robust API, and automated scraping capabilities to keep information synchronized.

## Architecture

The project is structured as a **Monorepo** using **Nx** and **Bun** for workspace management and performance.

### 1. Structure
- `apps/api`: Backend service built with **Fastify**, **Bun**, and **Mongoose**.
- `apps/web`: Frontend application built with **Vue 3**, **Vite**, **Bun**, and **Shadcn-vue**.
- `libs/shared-schema`: Shared **Zod** schemas and TypeScript types used by both frontend and backend to ensure data consistency.
- `docs/`: Project documentation and guidelines.

### 2. Frontend Stack
- **Framework**: Vue 3 (Composition API, `<script setup>`)
- **Styling**: Tailwind CSS + Shadcn-vue
- **State/Data Fetching**: TanStack Query (Vue Query) + Axios
- **Form Validation**: Zod (via shared-schema)
- **Icons**: Lucide-vue-next
- **Build Tool**: Vite (running on Bun)

### 3. Backend Stack
- **Runtime**: Bun
- **Framework**: Fastify
- **Database**: MongoDB (Mongoose ODM)
- **Validation**: Zod (via shared-schema)
- **Security**: @fastify/helmet, @fastify/cors

---

## Database & Data Models

### Shared Schema (`libs/shared-schema`)
All data structures are defined using **Zod** to provide both runtime validation and TypeScript types.

```typescript
// libs/shared-schema/src/index.ts
import { z } from 'zod';

export const AlumniSchema = z.object({
  firstName: z.string().min(2, "First name must be at least 2 characters"),
  lastName: z.string().min(2, "Last name must be at least 2 characters"),
  email: z.string().email("Invalid email address"),
  graduationYear: z.number().int().min(1900).max(new Date().getFullYear() + 10).optional(),
});

export type AlumniType = z.infer<typeof AlumniSchema>;
```

### Backend Models (`apps/api/src/models`)
Mongoose models use the types inferred from the shared Zod schemas.

#### Alumni Model
- **Collection**: `alumni`
- **Fields**:
  - `firstName` (String, Required)
  - `lastName` (String, Required)
  - `email` (String, Required, Unique)
  - `graduationYear` (Number)
  - `timestamps` (createdAt, updatedAt)

---

## Best Practices

### 1. Data Consistency
- **Always** use the `shared-schema` for any data that travels between the frontend and backend.
- Use `AlumniSchema.safeParse()` on the frontend before sending data.
- Use the same schema on the backend to validate incoming requests.

### 2. Development Workflow
- Use **Bun** for all commands (`bun install`, `bun dev`, `bun test`).
- Run the full stack locally using the root scripts:
  - `bun dev:api`
  - `bun dev:web`

### 3. Frontend Guidelines
- **Component-Driven**: Build reusable components in `src/components/ui` (Shadcn-vue).
- **Feature-Based**: Organize domain logic in `src/features/`.
- **Composition API**: Use `<script setup>` and composables for better logic reuse.
- **Visuals**: Maintain a clean, professional aesthetic using Shadcn-vue's design system.

### 4. Backend Guidelines
- **Surgical Updates**: When modifying the API, ensure all routes are properly typed and validated.
- **Fastify Plugins**: Use plugins for modularity (CORS, Helmet, etc.).
- **Error Handling**: Return clear, structured error messages (compatible with Zod issues if possible).

### 5. Scraping (Future)
- Use **Playwright** for automated data collection.
- Scraping logic should be isolated and follow the shared schema for data ingestion.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mheelb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mheelb)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
