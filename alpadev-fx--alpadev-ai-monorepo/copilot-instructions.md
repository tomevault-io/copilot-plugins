## alpadev-ai-monorepo

> **Name:** alpadev AI Monorepo

# alpadev AI Monorepo Context

## 1. Project Identity
**Name:** alpadev AI Monorepo
**Type:** Full-stack Monorepo (TurboRepo + pnpm)
**Core Purpose:** A scalable, microservices-ready platform integrating a Next.js frontend with AI, Web3, and WhatsApp automation capabilities.
**State:** Active Development / Refactoring

## 2. Tech Stack & Architecture

### **Monorepo Structure**
*   **Manager:** TurboRepo
*   **Package Manager:** pnpm
*   **Workspaces:** `apps/*`, `packages/*`

### **Frontend (`apps/frontend`)**
*   **Framework:** Next.js 15 (App Router)
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS, HeroUI, Framer Motion, GSAP
*   **3D/Graphics:** Three.js (`@react-three/fiber`, `@react-three/drei`)
*   **State/Data:** React Query, tRPC Client, Zustand (implied or commonly used with this stack)

### **Backend & Services**
*   **API Layer:** tRPC (`packages/api`) / Next.js API Routes
*   **Database:** MongoDB (via Prisma ORM in `packages/db`)
    *   *Note: README mentions PostgreSQL, but code configuration (Schema/Docker) confirms MongoDB.*
*   **Authentication:** NextAuth.js (`packages/auth`)
*   **Email:** React Email / Resend (`packages/email`)
*   **Validation:** Zod (`packages/validations`)
*   **Web3:** Hardhat, Solidity Contracts (`packages/web3`)
*   **Integrations:** Twilio (WhatsApp Webhooks)

### **Architectural Patterns**
*   **Clean Architecture:** Strict separation of concerns (Router → Service → Repository → DB) is enforced.
*   **Type Safety:** End-to-end type safety via tRPC and shared validation schemas.
*   **Atomic Design:** Frontend components organized for modularity and reusability.

## 3. Key Development Commands

| Action | Command | Description |
| :--- | :--- | :--- |
| **Start Dev** | `pnpm dev` | Starts all apps in development mode (parallel). |
| **Build** | `pnpm build` | Builds all apps and packages. |
| **Lint** | `pnpm lint` | Runs ESLint across the workspace. |
| **Format** | `pnpm format` | Formats code using Prettier. |
| **Clean** | `pnpm clean` | Removes `node_modules`, `.next`, and build artifacts. |

### **Database Operations (Prisma)**
*   `pnpm db:push` - Push schema changes to MongoDB.
*   `pnpm db:generate` - Generate Prisma Client.
*   `pnpm db:studio` - Open Prisma Studio to view data.
*   `pnpm db:seed` - Seed the database.

## 4. Key Directories

*   `apps/frontend`: Main user interface.
*   `packages/db`: Database schema (`schema.prisma`) and client configuration.
*   `packages/api`: Backend logic and tRPC routers.
*   `packages/web3`: Smart contracts and blockchain interactions.
*   `agents/`: System context, architectural protocols, and AI agent instructions.
*   `tickets/`: Project roadmap and active tasks.

## 5. Development Status & Roadmap
The project is currently undergoing a "Frontend Refactoring" phase (see `tickets/ROADMAP.md`) focusing on:
1.  **Performance:** FPS stability, bundle size optimization.
2.  **Quality:** Deduplication, TypeScript strict mode.
3.  **Architecture:** Atomic design implementation.

## 6. Critical Context & Rules
*   **Strict Typing:** `any` types are discouraged.
*   **Forbidden Patterns:**
    *   Repositories depending on other repositories.
    *   Business logic in Routers/Controllers.
    *   Direct database access from UI components (use tRPC/API).
*   **Source of Truth:** `packages/db/prisma/schema.prisma` is the definitive source for the data model (MongoDB).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alpadev-fx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alpadev-fx)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
