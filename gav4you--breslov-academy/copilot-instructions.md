## breslov-academy

> This file provides instructional context for Gemini AI agents working on the Breslov Academy LMS project. Adhere to these standards and architectural invariants for all code modifications and feature implementations.

# Breslov Academy - Gemini Instructional Context

This file provides instructional context for Gemini AI agents working on the Breslov Academy LMS project. Adhere to these standards and architectural invariants for all code modifications and feature implementations.

## Project Overview
Breslov Academy is a **multi-tenant white-label Learning Management System (LMS)** designed for high-trust educational content delivery.

- **Primary Technologies:** React (v19), Vite, Tailwind CSS, shadcn/ui, TanStack Query (v5), Lucide React, Three.js/React-Three-Fiber.
- **Backend/API:** Cloudflare-compatible edge API via `src/api/appClient.js` (Base44 SDK removed).
- **Repository:** `git@github.com:yeshivabachur/breslov-academy.git` (Main)

## Architecture (v11.0)
The application uses a **Portalized Architecture** split into distinct surfaces:
- **/ (Public):** Guest-safe marketing site (`src/pages/SchoolLanding.jsx`).
- **/student:** Learner experience, course progress, quizzes, and social features.
- **/teacher:** Content authoring (`TeachCourse`), grading, and analytics.
- **/admin:** School-level configuration (`SchoolAdmin`), monetization, and integrity.
- **/virtual-beit-midrash:** 3D immersive study environment (`src/components/vr/VirtualBeitMidrash.jsx`).
- **/ai-tutor:** Conversational AI interface (`src/components/ai/AITutorInterface.jsx`).

## Key Systems
1.  **Feature Registry:** A single source of truth (`src/components/config/features.jsx`) defines all features, routes, and audience access.
2.  **Feature Flags:** Dynamic toggling of features per school/user (`src/components/config/featureFlags.js`).
3.  **Integrations:** Centralized marketplace for third-party apps (`src/components/config/integrations.js`).
4.  **Tenancy Scoping:** Strictly enforced school-level isolation via `scopedFilter` and `tenancyEnforcer.js`.

## Development Conventions & Invariants

### 1. Registry-First Policy
**NEVER** add a route to `src/App.jsx` without first registering it in `src/components/config/features.jsx`.

### 2. Multi-Tenant Scoping (The "Scoped Rule")
All entities listed in `src/components/api/scopedEntities.js` are **school-scoped**.
- **PROHIBITED:** Direct calls to `base44.entities.<Entity>.*` for school data.
- **REQUIRED:** Use scoped helpers from `src/components/api/scoped.jsx`.

### 3. Content Protection (The "Locked Rule")
- **Hard Invariant:** If a lesson is `LOCKED` or `DRIP_LOCKED`, the application **must not** fetch premium materials.
- Use `useLessonAccess` hook before triggering content queries.

### 4. Design System
- Use tokens from `src/components/theme/tokens.js`.
- Prefer `GlassCard`, `PageShell`, and `SkeletonLoaders` over raw Tailwind classes.

### 5. Mobile Native
- Respect safe areas using `pt-safe-top` and `pb-safe-bottom` classes.
- Use `useIsNative()` hook for platform-specific logic.

## Build & Run
- `npm run dev`: Start development server.
- `npm run build`: Generate production build (`dist/`).
- `npm run lint`: strict linting.
- `npm run typecheck`: TypeScript verification.

## Integrity
Admins can visit `/integrity` to run automated scans for registry drift and data leakage.

---
> Source: [gav4you/breslov-academy](https://github.com/gav4you/breslov-academy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
