## schedule-wizard

> This file defines how to build, test, and evolve Schedule Wizard.

# Schedule Wizard Agent Guide

This file defines how to build, test, and evolve Schedule Wizard.

## 1. Project Overview

- Product: Schedule Wizard
- Goal: a voice-first iOS app for schedule management, todos, and expense tracking.
- Core UX: user speaks, types, or uploads an image; AI interprets one or more actions; the app shows a confirmation sheet; database writes happen only after user confirmation.
- Architecture: thin client. The client handles UI, native capabilities, and streaming display. The server owns business logic, AI orchestration, validation, persistence, and side effects.

## 2. Source of Truth

- Product behavior: [`PRD.md`](/Users/pierrewang/Documents/repos/schedule-wizard/PRD.md)
- Technical architecture: [`TECH_DESIGN.md`](/Users/pierrewang/Documents/repos/schedule-wizard/TECH_DESIGN.md)
- If this file conflicts with those documents, follow the PRD for product behavior and the tech design for architecture.
- If implementation reveals a real conflict between the PRD and tech design, document it explicitly and resolve it before building further.

## 3. Current Repository State

- The repository is still close to the default Ionic starter app.
- Existing `Tab1/Tab2/Tab3`, starter tests, and placeholder UI are scaffolding and should be replaced.
- `server/`, `shared/`, database schema, AI orchestration, and production-grade feature modules still need to be created.
- There is an existing `package-lock.json`; from this point forward the project standard is `pnpm`, not `npm`.

## 4. Required Stack

Use the stack defined in `TECH_DESIGN.md`, with the latest stable compatible versions at implementation time.

### Client

- Ionic + React for the iOS-first app shell
- Capacitor for native integrations
- React 19
- React Router v5 for Ionic compatibility unless the app is intentionally migrated with a validated replacement plan
- TanStack Query for server state
- Zustand only for UI-local state
- TypeScript with strict mode
- Vite for frontend builds

### Server

- Node.js current active LTS
- Hono for the API layer
- Drizzle ORM with PostgreSQL on Neon
- Clerk for authentication
- Cloudflare R2 for uploaded files
- Vercel AI SDK for streaming, structured output, and tools

### Package Management

- Use `pnpm` only.
- Create and maintain `pnpm-lock.yaml`.
- Do not introduce or update `package-lock.json`.
- When migrating existing dependencies from the starter app, prefer the latest stable compatible versions rather than copying outdated examples.

## 5. Non-Negotiable Product Rules

- AI parsing must not write directly to the database.
- All create, update, and delete actions proposed by AI must first become `PendingTask` items.
- Users must be able to confirm or cancel the proposed actions before they take effect.
- A single input may produce multiple tasks across domains.
- The client is responsible for recording, image capture, haptics, rendering, and streaming UX.
- The server is responsible for intent parsing, task planning, validation, persistence, and authorization.
- All user data access must be scoped by authenticated `userId`.

## 6. Implementation Priorities

Build in this order unless a dependency forces a change.

1. Replace starter tabs with the real application shell and routing.
2. Establish shared types and the backend project structure.
3. Implement auth plumbing and API client conventions.
4. Implement database schema, migrations, and core CRUD services.
5. Implement AI chat SSE flow and pending task storage.
6. Implement confirmation flow and database commit path.
7. Implement Today view, timeline views, calendar views, and day detail flows.
8. Add upload, receipt/image handling, reminders, and reporting enhancements.

## 7. Development Specifications

### Project Structure

Target structure should follow `TECH_DESIGN.md`:

- `src/`: Ionic React client
- `server/`: Hono API server
- `shared/`: shared models, DTOs, and validation schemas

Keep boundaries strict:

- UI components do not contain business rules that belong on the server.
- API route handlers stay thin and delegate to services.
- Shared types must not import client-only or server-only modules.

### State Management

- TanStack Query is the default for any server-backed data.
- Zustand is only for ephemeral UI state such as recording state, current view mode, active sheet, and conversation UI state.
- Do not duplicate server state in Zustand.

### Validation and Types

- All API inputs and outputs should have runtime validation and static TypeScript types.
- Prefer a shared schema-first approach for request and response contracts.
- Never trust AI output, client payloads, or uploaded file metadata without validation.

### AI Flow

- Use structured outputs for orchestration and tool inputs.
- Domain agents should be constrained to explicit tools and validated payload shapes.
- Log enough metadata to debug failed task generation, but never log secrets or raw auth tokens.
- Handle ambiguity with clarification instead of guessing when the action could be wrong or destructive.

### Data and Time

- Store timestamps with timezone support.
- Be explicit about timezone handling across client, server, and AI prompts.
- Currency defaults to `CNY` unless the product requirement changes.
- Use decimal-safe handling for money. Do not rely on floating-point math for persisted finance logic.

### Mobile UX

- Optimize for iOS-first behavior and feel.
- Keep the bottom input bar persistent across primary views.
- Preserve scroll position when switching timeline/calendar views where possible.
- Recording interactions need immediate visual and haptic feedback.

## 8. Testing Requirements

Every meaningful feature change must include tests or a clear written reason why testing is not practical yet.

### Minimum Coverage Expectations

- Shared schemas and pure utilities: unit tests
- Server services and orchestrator logic: unit tests
- API routes: integration tests
- Critical user journeys: end-to-end tests

### Must-Test Areas

- Event and transaction validation
- Auth enforcement and user scoping
- Pending task lifecycle: create, fetch, confirm, cancel, expire
- AI orchestration for single-domain and multi-domain inputs
- Confirmation-before-write guarantee
- SSE streaming behavior and error handling
- Timezone-sensitive parsing and date-range queries
- Optimistic updates with rollback on failure
- File upload validation and access control

### Testing Standards

- Do not keep the starter smoke tests as the only coverage.
- Replace placeholder Cypress tests with real flows.
- Add regression tests for every bug fix that affects logic, state transitions, or API behavior.
- Prefer deterministic tests with mocked AI responses and fixed clocks.
- Avoid tests that depend on live third-party services.

### Expected Commands

- Install: `pnpm install`
- Frontend dev: `pnpm dev` (don't run this unless I told you to do so)
- Server dev: `pnpm dev:server`
- Frontend build: `pnpm build`
- Server build: `pnpm build:server`
- Lint: `pnpm lint`
- Unit tests: `pnpm test.unit`
- E2E tests: `pnpm test.e2e`

Add missing scripts as the server and workspace are created. Keep command names consistent with this guide unless there is a strong reason to change them.

## 9. Code Style

- Language: TypeScript everywhere possible.
- Use strict typing. Do not weaken types with `any` unless there is a narrow, documented boundary.
- Prefer small focused modules over large mixed-responsibility files.
- Keep React components presentational where possible; move data fetching and orchestration into hooks or feature modules.
- Favor explicit function names over comments that restate the code.
- Use descriptive domain names: `event`, `transaction`, `pendingTask`, `confirmationId`.
- Keep side effects isolated.
- Use early returns and simple control flow.
- Do not introduce hidden global state.

### React Conventions

- Prefer function components and hooks.
- Keep page-level data fetching near page hooks, not scattered across leaf components.
- Avoid unnecessary memoization. Add it only when there is a measured reason.
- Respect Ionic navigation patterns and page lifecycle behavior.

### Server Conventions

- Route handlers validate input, extract auth context, and call services.
- Services contain domain logic.
- Database access stays centralized and typed.
- Errors returned to clients must be safe, structured, and consistent.

## 10. Dependency Policy

- Prefer the latest stable release of each library at the time of installation.
- Verify peer dependency compatibility before adding or upgrading packages.
- Do not pin obsolete versions copied from blog posts or templates.
- Keep client and server libraries aligned with the chosen platform versions, especially Ionic, Capacitor, React, Clerk, Drizzle, and the AI SDK.
- Remove unused starter dependencies when they no longer serve the target architecture.

## 11. Precautions

- Do not bypass the confirmation step for AI-created mutations.
- Do not put server business logic into the mobile client.
- Do not trust image OCR, speech transcription, or LLM output without validation.
- Do not leak cross-user data in queries, logs, caches, or pending-task storage.
- Do not make finance calculations with imprecise number handling.
- Do not ship placeholder routes, starter tab content, or mock data as production behavior.
- Do not let optimistic updates become the source of truth; always reconcile with server state.
- Do not introduce breaking schema or API changes without updating shared contracts and tests together.
- Do not keep both `npm` and `pnpm` lockfiles in active use.
- Do not run `git commit` (or `git push`) until the user explicitly tells you to commit. Staging and drafting messages is fine; creating the commit is not.

## 12. Definition of Done

A task is not complete unless all of the following are true:

- Implementation matches the PRD and tech design, or the deviation is documented and approved.
- Types, validation, and error handling are in place.
- Necessary tests were added and pass locally.
- Lint and build pass locally.
- Any new dependency uses `pnpm` and a current stable compatible version.
- Placeholder starter code related to the changed area has been removed or replaced.
- Important assumptions, follow-ups, and tradeoffs are documented in the change summary or relevant docs.

## 13. First Major Refactor Checklist

Before deeper feature work, complete these baseline tasks:

- Replace Ionic starter tabs and starter tests.
- Introduce `server/` and `shared/` directories.
- Add `pnpm-lock.yaml` and standardize scripts around `pnpm`.
- Add backend dev/build scripts from the technical design.
- Add validation/schema infrastructure shared across client and server.
- Set up a test foundation for unit, integration, and e2e coverage.

---
> Source: [jianghuawang/schedule-wizard](https://github.com/jianghuawang/schedule-wizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
