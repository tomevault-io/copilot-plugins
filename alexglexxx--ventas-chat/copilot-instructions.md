## ventas-chat

> This document contains foundational mandates for the Ventas-Chat project. These instructions take absolute precedence over general workflows and tool defaults.

# Project Mandates: Ventas-Chat

This document contains foundational mandates for the Ventas-Chat project. These instructions take absolute precedence over general workflows and tool defaults.

## 1. Technical Stack & Conventions

- **Next.js 16 (App Router):** This project uses Next.js 16. Refer to `AGENTS.md` for breaking changes. Follow App Router conventions strictly.
- **React 19:** Adhere to React 19 patterns and best practices.
- **Firebase Core:** Use Firebase (Auth, Firestore, Hosting) for real-time features and authentication.
- **Firebase Data Connect:** Primary data layer for structured relational data (PostgreSQL/Cloud SQL). Configuration in `dataconnect/dataconnect.yaml`. Connector definitions in `dataconnect/example/`.
- **Firestore:** Used for real-time messaging and transient data (see `components/Chat.tsx`).
- **Coexistence:** Use Data Connect for formal business entities (Users, Businesses, Menus, Orders) and Firestore for live chat interactions.
- **TypeScript:** Strict typing is mandatory. No `any` unless explicitly required by a third-party library. Use generated types from `@dataconnect/generated`.
- **Styling:** Tailwind CSS 4. Maintain the minimalist, high-contrast aesthetic (Black, White, Zinc).

## 2. Architecture & Directory Structure

- **App Router (`app/`):** All pages and layouts must be within the `app` directory. Use dynamic routes like `[businessId]` for business-specific pages.
- **Components (`components/`):** UI components should be modular and reusable.
- **Lib (`lib/`):** Shared utilities and initializations (e.g., `firebase.ts`).
- **Data Connect (`dataconnect/`):** Schema and operations for Data Connect.
- **Generated Code (`src/dataconnect-generated/`):** Do not modify generated files manually.

## 3. Engineering Standards

- **Real-time Sync:** Prefer `onSnapshot` for features requiring real-time updates (like the chat).
- **Security Rules:** Always consider `firestore.rules` and Data Connect authorization when adding new features.
- **Performance:** Optimize for low-bandwidth environments (common in street commerce settings).
- **Consistency:** Follow existing naming conventions and component structures (see `components/Chat.tsx` and `app/page.tsx`).

## 4. Visual Identity

- **Theme:** Clean, professional, and accessible. Use the Zinc color palette for neutral elements.
- **Typography:** Sans-serif (Geist/Inter) with tight tracking for headings.
- **Interactions:** Subtle, smooth transitions and hover states.

## 5. Development Workflow

- **Tests:** Always update or add tests in the `test/` directory for new features or bug fixes.
- **Environment:** Respect the local environment setup and Firebase configuration.
- **AI Logic:** For AI-enhanced features, refer to the `firebase-ai-logic` and `developing-genkit-js` skills.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexglexxx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
