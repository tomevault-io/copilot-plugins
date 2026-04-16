## gast-ai

> This document provides a comprehensive overview of the `Gast.ai` project. It serves as the source of truth for architectural decisions, coding standards, and project goals for AI-powered development assistance.

# GEMINI.md - Project Overview: Gast.ai (gastos-kv-mvp)

This document provides a comprehensive overview of the `Gast.ai` project. It serves as the source of truth for architectural decisions, coding standards, and project goals for AI-powered development assistance.

## Project Overview

`Gast.ai` (formerly gastos-kv-mvp) is a Next.js application for tracking expenses. The application has successfully transitioned its core data layer from a personal tool using Vercel KV to a multi-user SaaS platform using PostgreSQL. The primary interface includes a voice-first page for recording expenses via Google Gemini AI.

### Core Technologies

*   **Framework:** Next.js (React - App Router)
*   **Language:** TypeScript
*   **Database:** Neon (Serverless PostgreSQL)
*   **ORM:** Prisma (v7+) with `@prisma/adapter-pg` and `pg` pool.
*   **Authentication:** NextAuth.js (Auth.js) with Credentials & bcryptjs
*   **Validation:** Zod
*   **Styling:** PandaCSS (zero-runtime, type-safe CSS-in-JS)
*   **AI Engine:** Google Gemini SDK (Natural language parsing of expenses)
*   **Mobile / PWA:** next-pwa (Installable web app logic)

### Architecture (Clean Code & Separation of Concerns)

The project strictly follows a layered architecture to separate business logic from UI and infrastructure:

*   **1. Presentation Layer (UI/Components/PWA):**
    *   `/app`: Next.js App Router pages, API route handlers, and PWA manifest.
    *   `/components`: Reusable UI components following Atomic Design (`ui`, `features`, `layout`). Includes real-time UX feedback, animations, and dynamic financial summaries for voice-first interactions.
*   **2. Application / Validation Layer:**
    *   `/schemas`: Zod schemas for strict runtime validation of API requests and form submissions (e.g., `expenseSchema.ts`, `authSchema.ts`). All data must pass Zod before reaching the Repository.
*   **3. Business Logic Layer (AI Integration):**
    *   `/services`: Contains core business rules. Specifically, `aiParserService.ts` replaces legacy Regex functions by calling the Google Gemini API (`gemini-2.5-flash-lite`).
        *   **Parser:** Extracts structured JSON (`amountCents`, `description`, `date`, `paymentMethod`) with strict low temperature (`0.3`) for precision. **Crucial:** Always passes the client's local timezone (`x-timezone`) to Gemini to ensure expenses near midnight resolve to the correct local day, overriding the server's default UTC context.
        *   **Analyzer (Gastão Persona):** Provides a streaming natural language analysis using an even lower temperature (`0.1`) to guarantee deterministic math while maintaining a sarcastic, friendly persona ("Gastão"). Prompts explicitly forbid markdown styling for a cleaner UI render.
*   **4. Data Access Layer (Repository Pattern):**
    *   `/repositories`: Abstracts all database operations. API routes must call repositories (e.g., `ExpenseRepository.create`), **never Prisma directly**. This makes future database migrations trivial.
    *   `/prisma`: Contains `schema.prisma` defining `User` and `Expense` models. **Note:** Prisma v7 uses `prisma.config.ts` for database connection URLs, not the schema file.
    *   `/lib/prisma.ts`: The singleton instance of Prisma. It uses `pg` Pool and `@prisma/adapter-pg` to ensure compatibility and performance in Serverless environments. The Prisma Client is generated into a custom path (`/lib/generated/client/client`).

## Development Conventions & Current State

*   **Database Access:** 
    *   **NEVER** use direct ORM calls (`prisma.expense...`) inside Next.js API Routes (`app/api/.../route.ts`).
    *   **ALWAYS** use the Repository Pattern (e.g., `ExpenseRepository.create(...)`).
*   **Validation:** All incoming API payloads must be validated against a Zod schema before processing.
*   **Authentication:** The system uses NextAuth.js for session management with a custom Credentials provider and bcryptjs for secure password hashing. APIs are protected using `getServerSession`.
*   **Error Handling:** API endpoints must return structured JSON errors (`{ error: string }`) with appropriate HTTP status codes (400 for validation, 401 for auth, 500 for server errors).
*   **Code Style:** Strict TypeScript. No `any`. Variables must be camelCase, components PascalCase.

## Key Commands

*   `npm run dev` - Start development server
*   `npx prisma db push` - Sync Prisma schema with database (Neon)
*   `npx prisma generate` - Generate the Prisma Client to the custom `/lib` directory
*   `npx prisma studio` - Open Prisma visual database editor locally
*   `npx tsc --noEmit` - Type-check the project

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodrigojosebr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
