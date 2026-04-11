## donasai

> This file serves as the primary directive for the AI agent (Antigravity/GEMINI) when working on the `donasai` project.

# GEMINI.md - Project Context & Rules

This file serves as the primary directive for the AI agent (Antigravity/GEMINI) when working on the `donasai` project.

## 1. Documentation Authority
You MUST strictly follow the documentation in `wp-content/plugins/donasai/docs/`. These documents are the single source of truth:

*   **[prd.md](file:///Users/hsi/Dev/pribadi/donasai/wp-content/plugins/donasai/docs/prd.md)**: Product Requirements Document. Defines scope, features (Free vs Pro), and non-goals.
*   **[technical-specs.md](file:///Users/hsi/Dev/pribadi/donasai/wp-content/plugins/donasai/docs/technical-specs.md)**: Technical Specifications. Defines:
    *   File & Folder Structure
    *   Database Schema (tables: `donasai_donations`, `donasai_campaign_meta`)
    *   Technology Stack (PHP 7.4+, React + Vite + TS for Admin, Tailwind)
    *   API Endpoints
    *   Code Standards
*   **[user-story.md](file:///Users/hsi/Dev/pribadi/donasai/wp-content/plugins/donasai/docs/user-story.md)**: User Stories & Sprint Priorities.

## 2. Critical Rules
1.  **Do NOT deviate** from the folder structure defined in `technical-specs.md`.
2.  **Do NOT deviate** from the database schema defined in `technical-specs.md`.
3.  **Prioritize** features based on "Sprint 1: Core Campaign + Form (Free MVP)" in `user-story.md`.
4.  **Admin UI**: Must use React + TypeScript + shadcn UI, built with Vite (`admin-app/`).
5.  **Frontend**: Use PHP templates + Tailwind CSS (`frontend/`).
6.  **Code Quality**: Follow WordPress Coding Standards and best practices mentioned in `technical-specs.md` (Security, Sanitization, Nonces).

## 3. Workflow
*   Always reference the above documents before starting a new task to ensure alignment.
*   Update `task.md` to track progress against the specific acceptance criteria in `user-story.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dankerizer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dankerizer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
