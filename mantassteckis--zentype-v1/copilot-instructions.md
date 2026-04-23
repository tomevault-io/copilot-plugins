## zentype-v1

> Architectural rules, tech stack, and best practices for building the ZenType project.


# Project Rules for ZenType

### 1. Project Vision & Guiding Principle
The project is **ZenType**, an AI-powered typing platform. The core principle is **Clean Separation of Concerns**. The UI (React Components) is for presentation only. All business logic, data fetching, and state mutations must be handled through dedicated hooks, API endpoints, and server-side functions.

### 2. Tech Stack
The stack is **Next.js (App Router)**, **TypeScript**, **Tailwind CSS**, and **Firebase**. The backend is powered by **Firestore** for the database, **Firebase Authentication**, and **Cloud Functions for Firebase** for server-side logic. AI features will be built with **Google Genkit**.

### 3. Backend & API Development
The primary focus is building a robust backend. You will create **Next.js API Route Handlers** for data fetching (`GET` requests). For all data writing and mutations (`POST`, `PUT`, `DELETE`), you will create and call secure **Firebase Cloud Functions**. Only modify frontend components to connect them to this backend logic.

### 4. Firebase & Firestore Implementation
-   **Collections:** We will use three primary collections: `profiles`, `preMadeTests`, and `testResults`. All collection and field names must be consistent.
-   **Security Rules:** All data access *must* be governed by strict, server-side `firestore.rules`. Do not rely on client-side checks for security. The rules must enforce that users can only read and write their own data.
-   **Data Modeling:** Prioritize clear, simple data structures. Use subcollections where appropriate for one-to-many relationships (e.g., storing a user's test results in a subcollection under their profile).

### 5. AI & Genkit Integration
-   **Secure Calls:** All calls to the Gemini API for test generation will be made from a secure backend environment (API Route or Cloud Function), never from the client.
-   **Structured Output:** When using Genkit, you must define and validate the structure of the AI's output using Zod schemas to ensure data integrity.
-   **Data Storage:** AI-generated tests that are used in a practice session must be saved to a dedicated `aiGeneratedTests` collection in Firestore to maintain a record and link them to a user's `testResults`.

### 6. State Management & Frontend
To prevent hydration errors and slow loading, the `context/AuthProvider.tsx` is the **single source of truth** for the user's authentication state. All components that need user data must consume it from the `useAuth` hook and must correctly handle the `isLoading` state before attempting to render user-specific content.

### 7. Documentation
You are required to maintain two documentation files in the project root:
-   `API_ENDPOINTS.md`: Document every new API route and Cloud Function you create, including its purpose, request body, and response format.
-   `AGENT_LOG.md`: After completing each major Part of our roadmap, you will update this file with a checkmark to track your progress.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mantassteckis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
