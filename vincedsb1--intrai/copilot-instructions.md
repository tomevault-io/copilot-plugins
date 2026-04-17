## intrai

> *   **Langue :** Toujours répondre en **Français**.

# GEMINI.md - Context & Instructions

## Règles Opérationnelles (Strictes)
*   **Langue :** Toujours répondre en **Français**.
*   **Git :** Ne JAMAIS effectuer de `commit` ou `push` sans demande explicite de l'utilisateur.
*   **Logs :** Ne JAMAIS supprimer de fichiers de logs sans demande explicite.

## Project Overview
**Name:** intrai (Job Aggregator & Single-Stream Hub)
**Purpose:** A streamlined web application for managing job offers. It features a "Single Stream" inbox, quick decision-making workflows (Save/Trash), automated filtering (Whitelist/Blacklist), and AI-powered author analysis ("AI Detective").
**Current Status:** Specification & Documentation phase. The codebase has not yet been initialized.

## Technology Stack
*   **Framework:** Next.js (App Router)
*   **Language:** TypeScript (Strict mode)
*   **Styling:** Tailwind CSS v4.1, Lucide React (Icons)
*   **Database:** MongoDB
*   **Testing:** Vitest (Unit & Integration)
*   **Deployment:** Vercel

## Architecture
The project follows a modular structure (monorepo style):
*   `/app`: Next.js App Router pages and API routes (`/app/api/**`).
*   `/components`: Reusable UI components (JobCard, TabsNav, etc.).
*   `/lib`: Database connections (MongoDB singleton), validation, helpers.
*   `/server`: Business logic (ingestion sorting, rules, AI detective).
*   `/docs`: Project documentation.
    *   `/docs/*.md`: Core project specifications (numbered).
    *   `/docs/rules/*.md`: Tech-specific coding rules (React, Next.js, etc.).
    *   `/docs/meta/*.md`: Documentation templates and guides.
*   `/tests`: Vitest test suites.

## Core Concepts & Data Model
*   **Job Entities:**
    *   **Status:** `INBOX` | `SAVED` | `TRASH`
    *   **Category:** `TARGET` (Whitelist) | `EXPLORE` (Default) | `FILTERED` (Blacklist)
    *   **Visited:** UI-state (grayed out), potentially persistent.
*   **AI Detective:** Analyzes job authors to identify spam/irrelevant sources.

## Getting Started (Implementation Plan)
Since the project is currently just documentation, the immediate next steps for an agent would be:
1.  **Initialize Project:** Run `npx create-next-app@latest` (or similar) with TypeScript, Tailwind, and App Router.
2.  **Scaffold Structure:** Create the folders defined in `ARCHITECTURE.md` (`/components`, `/lib`, `/server`, etc.).
3.  **Setup Database:** Configure MongoDB connection in `/lib/mongo.ts`.
4.  **Implement Core Logic:** Start with `server/jobs.service.ts` and `server/settings.service.ts`.

## Documentation Reference
*   `ARCHITECTURE.md`: High-level architectural decisions and current state.
*   `docs/00_INDEX.md`: Main index and reading order.
*   `docs/MAQUETTE_UI.tsx`: Interactive UI Prototype (React + Tailwind).
*   `docs/01_OBJECTIF_PRODUIT.md`: Product vision and UX principles.
*   `docs/12_CONVENTIONS_DEV.md`: Coding standards and naming conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vincedsb1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
