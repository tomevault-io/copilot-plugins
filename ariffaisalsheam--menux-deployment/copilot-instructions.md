## menux-deployment

> This document defines how AI agents and developers should operate within the Menu.X codebase, tools, and environments. It encodes security, coding, validation, API, MCP usage, and deployment rules to ensure safe and consistent changes.


# Menu.X AI Rules (AI Operating Guide)

This document defines how AI agents and developers should operate within the Menu.X codebase, tools, and environments. It encodes security, coding, validation, API, MCP usage, and deployment rules to ensure safe and consistent changes.

---

## 1) Scope & Roles

- __Project__: Menu.X – digital restaurant platform for Bangladesh.
- __Primary roles__: Diner (anon), Restaurant Owner, Super Admin.
- __Repos/Dirs__:
  - Backend (Spring Boot, Java 17): `backend/`
  - Frontend (React + Vite + TS): `frontend/`
  - Docs and scripts at repo root.

---

## 2) Tech Stack & Key Paths

- __Backend__: Spring Boot, PostgreSQL (H2 local), Flyway, Spring Security + JWT, Maven.
  - Config: `backend/src/main/resources/application.yml`
  - Controllers: `backend/src/main/java/com/menux/menu_x_backend/controller/`
  - Services: `backend/src/main/java/com/menux/menu_x_backend/service/`
  - Security: `backend/src/main/java/com/menux/menu_x_backend/config/SecurityConfig.java`
- __Frontend__: React 19 + TS, Vite, Tailwind + shadcn/ui, axios, react-router.
  - Services: `frontend/src/services/api.ts`
  - Components: `frontend/src/components/`
  - Pages: `frontend/src/pages/`
- __Backends URLs__:
  - Production backend base URL: https://menux-backend-api.onrender.com

See `CODEBASE_INDEX.md` for a detailed backend index.

---

## 3) Security & Secrets

- __Never__ commit secrets. Do not hardcode API keys; use environment variables.
- Backend secrets come from the platform (Render) as environment variables, overriding `application.yml`.
- Media service keys are server-side only; the frontend must never receive service keys.
- Cursorily validate all inputs both client-side and server-side. Backend remains the source of truth.
- Public endpoints: GET `/api/media/stream` is intentionally public for <img> usage.

---

## 4) API Contracts & Conventions

- __AI Description Generation__
  - Frontend: `aiAPI.generateDescription(itemName: string)`
  - Backend: `AIController.generateMenuDescription` expects body `{ itemName }` and returns `{ description }`.
- __Menu Items__
  - Create via `menuAPI.createMenuItem(payload)`.
  - Required fields: `name` (non-empty), `price` (number > 0), `category` (enum), `isAvailable` (boolean), optional `description`, `imageUrl` as storage path.
- __Media__
  - Upload: `POST /api/media/upload` (multipart). Returns `{ path, proxyUrl }`.
  - Display: Use `mediaProxyUrl(path)` to render images via `/api/media/stream?path=...`.
- __Errors__
  - Prefer JSON error bodies with `error` or `message` fields. Show user-friendly messages on the frontend.

---

## 5) Frontend Rules

- __Validation first__: Validate required fields on the client before calling backend APIs.
  - In `AIMenuWriter.tsx`, block bulk save when any item has invalid `name`, `price`, or `category`.
  - Show inline field errors and a clear validation summary banner.
- __Forms__: Keep `price` as a string in inputs; convert to float only for payload.
- __Images__: Store returned `path` in state; build preview using `mediaProxyUrl(path)`.
- __Routing__: SPA rules via `vercel.json` catch-all to `/index.html`.
- __UI/UX__: Use shadcn/ui components. Avoid blocking operations in UI threads; show progress states.

---

## 6) Backend Rules

- __Validation__: Enforce DTO validation for create/update (e.g., price > 0, required fields). Return 400 with descriptive errors.
- __Ownership & Auth__: Controllers must verify restaurant ownership for protected resources.
- __Images__: When image URL changes or item deleted, ensure old storage objects are removed (implemented in `MenuController`).
- __Media Security__: Service key usage remains server-side (e.g., `MediaStorageService`). Handle URL encoding correctly and consistently.

---

## 7) AI Menu Builder Validation (Authoritative)

- Required per item before save:
  - __Name__: non-empty string.
  - __Price__: valid number > 0.
  - __Category__: must be selected (use backend-defined categories).
- __UI Behavior__:
  - Field-level errors near each input.
  - Validation summary banner when any item invalid; disable/block bulk save.
  - Clear field error when the user edits the corresponding field.
- __Payload mapping__:
  - `price: parseFloat(priceString)` before calling backend.

---

## 8) Media & Images

- Upload through `mediaAPI.uploadImage(file, restaurantId)`.
- Preview/display using `/api/media/stream?path=...` (no auth required for GET).
- File checks: Enforce reasonable client-side validation (e.g., jpeg/png/webp).

---

## 9) MCP Servers & When to Use

Available in this workspace: `supabase-mcp-server`, `sequential-thinking`, `puppeteer`, `mcp-playwright`.

- __mcp-playwright__
  - Purpose: UI navigation/smoke checks in a browser-like context.
  - Examples: take screenshots, wait for text, click elements, type, select options, open/close tabs.
  - Notes: Prefer non-destructive interactions; do not modify page content.
- __puppeteer__
  - Purpose: Scripted browser automation for DOM interactions.
  - Examples: navigate, click, fill inputs, select dropdowns, evaluate JS, capture screenshots.
  - Notes: Keep actions idempotent; avoid destructive flows unless explicitly requested.
- __sequential-thinking__
  - Purpose: Complex planning, iterative reasoning, and hypothesis revision before large edits.
  - Examples: break down tasks, generate and verify solution hypotheses, revise steps as needed.
  - Notes: Stop and reassess when new information contradicts prior assumptions.
- __supabase-mcp-server__
  - Purpose: Safe database visibility and controlled schema changes.
  - Safe reads: list tables, list migrations, list extensions, get advisors (security/performance), execute read-only SQL.
  - Migrations: apply DDL via migrations only; never hardcode generated IDs in data migrations; confirm intent before destructive ops.
  - Runtime: fetch project URL/API URL and anon key when needed; deploy edge functions with explicit file lists and import maps.

__General MCP rules__:
- Prefer read-only or non-destructive actions first.
- Clearly state why each tool is used and stop when dependent outputs are needed.
- Do not perform bulk destructive operations. Seek confirmation for any irreversible change.

---

## 10) Commands & Safety (Local/CI)

- Treat shell commands as potentially destructive; do not auto-run anything that installs, deletes, or mutates state without explicit approval.
- For long-running dev servers, run non-blocking and stream output if needed; stop when not required.

---

## 11) Testing & QA

- __Frontend__
  - Validate error states appear correctly (inline + summary) before save.
  - Confirm valid items save; invalid ones don’t trigger API calls.
  - Verify images upload and preview correctly via `/api/media/stream`.
- __Backend__
  - Unit/service tests for validation and ownership checks.
  - Ensure 400s include helpful messages.

---

## 12) Deployment Rules

- __Backend (Render)__
  - Dockerized Spring Boot. On push, Render builds with `backend/Dockerfile` and `pom.xml`.
  - Environment variables configured in Render dashboard override `application.yml`.
- __Frontend (Vercel)__
  - Root for frontend: `frontend/`
  - Build: `npm install` -> `tsc -b && vite build` -> deploy `dist/`.
  - SPA routing: `frontend/vercel.json` routes to `/index.html` for client-side routing.

---

## 13) Performance & Resilience

- Use retry/backoff for AI calls where appropriate; avoid hammering providers.
- Avoid parallelizing requests that may hit rate limits; sequential where required (e.g., Generate All).
- Keep payloads minimal; trim strings before submit.

---

## 14) Coding Standards

- Keep imports at top; no mid-file import additions.
- Keep edits minimal and contextual; include at least 3 lines of context when patching.
- Avoid committing large binaries or generated artifacts.
- Prefer small, focused PRs with clear summaries.

---

## 15) Error Handling & UX

- Always display friendly, actionable messages to the user.
- Log precise technical details to console/server logs; don’t expose internals to diners.
- In AI flows, surface retry attempts and allow manual retry.

---

## 16) Documentation & Memories

- Add/update README, `DEVELOPMENT_SETUP.md`, and `DEPLOYMENT.md` when workflows change.
- Use workspace memories to store:
  - Live endpoints and base URLs
  - Deployment flows
  - Key architectural constraints

---

## 17) What Not To Do

- Always ask if you're confused about something
- Don't pretent anything
- Don’t save menu items with missing name/price/category.
- Don’t expose Supabase service keys to the frontend.
- Don’t bypass ownership checks.
- Don’t push breaking schema changes without migrations and review.
- Don’t auto-run destructive commands or apply DB migrations without confirmation.

---

## 18) Quick References

- __AI__: `aiAPI.generateDescription(name)` -> `{ description }`.
- __Media__: Upload -> `path`; display via `mediaProxyUrl(path)` -> `/api/media/stream?path=...`.
- __Menu__: `menuAPI.createMenuItem({ name, description, price, category, isAvailable, imageUrl })`.

This document should be kept up to date as the platform evolves. When in doubt, prioritize security, validation, and user experience.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ariffaisalsheam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
