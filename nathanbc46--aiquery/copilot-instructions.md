## aiquery

> You are a Senior Full-stack Developer and AI Architect specializing in Nuxt 3 (Nuxt 4 compatibility mode), AI Integration (Text-to-SQL), and CRM Systems. You are building a middleware application that allows non-technical users to query Vtiger CRM 8.4 data using natural language, with a built-in manager approval workflow.

# Project Context & Rules: CRM AI-Query & Approval System

You are a Senior Full-stack Developer and AI Architect specializing in Nuxt 3 (Nuxt 4 compatibility mode), AI Integration (Text-to-SQL), and CRM Systems. You are building a middleware application that allows non-technical users to query Vtiger CRM 8.4 data using natural language, with a built-in manager approval workflow.

## Tech Stack & Architecture
- **Framework:** Nuxt 3.13.2 with `future: { compatibilityVersion: 4 }`, Nitro server, Auto-imports.
- **Styling:** Tailwind CSS (`@nuxtjs/tailwindcss`). Color mode (dark/light) via `@nuxtjs/color-mode`.
- **State Management:** Pinia 2.3.x + `@pinia/nuxt` 0.9.x.
- **PWA:** `@vite-pwa/nuxt` — app name "Vtiger AI Query System", Thai language.
- **Database:** MySQL (Direct connection to Vtiger CRM 8.4 Open Source).
- **ORM:** Drizzle ORM — schema at `server/utils/schema.ts`, migrations at `server/database/migrations/`.
- **AI Integration:** Google Gemini API (`@google/generative-ai`) for Text-to-SQL generation.
- **Email:** Nodemailer for approval notifications.
- **Export:** Zoho WorkDrive OAuth2 integration for document exports.

## Directory Structure
```
app/
├── pages/            → frontend pages (index, login, dashboard, approvals, history, admin/*)
├── components/       → ThemeToggle, ToastNotification
├── composables/      → useToast.ts
├── layouts/          → default.vue
└── middleware/       → auth.global.ts

server/
├── api/
│   ├── ai-query/     → generate, preview, analyze, refine, chat, request, approvals, action, export, export-zoho, history, report-error
│   ├── auth/         → login, logout, me
│   ├── admin/        → ai-settings, mail-settings, users, zoho-status, zoho-refresh
│   ├── users/        → CRUD
│   ├── favorites/    → save/list/delete named queries
│   ├── zoho/         → auth.get, callback.get (OAuth flow)
│   └── ...           → dashboard, data-guide, system-config, system/status
├── utils/            → db.ts, auth.ts, schema.ts, constants.ts, mail.ts, zoho.ts, rateLimit.ts
└── tasks/            → cleanup-snapshots.ts (runs daily at midnight)

brain/                → AI session memory
.data/snapshots/      → runtime snapshot storage
```

## Environment Variables (required)
```
DATABASE_URL          # MySQL connection string
GEMINI_API_KEY        # Google Gemini API key
SESSION_PASSWORD      # Session encryption key (min 32 chars)
APP_URL               # Application base URL
ZOHO_CLIENT_ID        # Zoho OAuth client ID
ZOHO_CLIENT_SECRET    # Zoho OAuth client secret
ZOHO_REDIRECT_URI     # Zoho OAuth redirect URI
ZOHO_FOLDER_ID        # Zoho WorkDrive target folder
```

## Core Logic Requirements
1. **AI Text-to-SQL Engine:**
   - Must use a "RAG for Schema" approach: Provide specific Vtiger table structures (Accounts, Contacts, Products, SalesOrder) in the system prompt via `data-guide.get.ts`.
   - AI must generate **Read-Only** SELECT queries only.
   - AI must explain the generated SQL logic in Thai so the manager understands what they are approving.
2. **Two-Step Execution (Safe-Run):**
   - **Step 1 (Preview):** Execute a `COUNT(*)` query first to show the number of results to the user/manager.
   - **Step 2 (Export):** Execute the full query only after manager approval.
3. **Approval Workflow:**
   - Requesting User: Submits query → System previews count → Clicks "Request Approval".
   - Approver (Manager): Receives request → Reviews SQL & Result count → Clicks "Approve".
   - Execution: System generates a downloadable CSV or JSON, optionally exported to Zoho WorkDrive.
4. **Vtiger Compatibility:** Handle complex joins between `vtiger_crmentity` and module-specific tables (e.g., `vtiger_account`, `vtiger_products`). Products are linked through `vtiger_inventoryproductrel` or custom module tables.
5. **Additional Features:**
   - **Favorites:** Users can save and reuse named queries (`/api/favorites/`).
   - **Query Refinement & Chat:** Users can refine generated SQL or chat with the AI (`refine.post.ts`, `chat.post.ts`).
   - **Error Reporting:** Frontend errors are reported via `report-error.post.ts`.

## Security & Guardrails
- **SQL Injection Prevention:** Use AI to validate that no `DROP`, `DELETE`, `UPDATE`, `INSERT`, or `TRUNCATE` commands exist in the generated SQL.
- **Database User:** Use a database user with **SELECT-only** permissions.
- **Rate Limiting:** Applied via `server/utils/rateLimit.ts` — do not remove or bypass.
- **Logging:** Every generated SQL and approval action must be logged for audit trails.
- **Privacy:** Ensure sensitive data (passwords, system configs) are excluded from the AI's schema context.

## Auth & SSR Patterns (Critical)
- **Middleware** (`app/middleware/auth.global.ts`): ใช้ `useRequestFetch()` เท่านั้น — ไม่ใช้ `useFetch` (cached) หรือ `$fetch` (ไม่ forward cookies server-side).
- **Layout auth fetch** (`app/layouts/default.vue`): ใช้ `useFetch` **ไม่มี `await`** + `useRequestHeaders(['cookie'])` เพื่อป้องกัน async Suspense hydration mismatch.
- **Auth-dependent `v-if`** ที่อาจต่างกันระหว่าง server/client ให้ครอบด้วย `<ClientOnly>` เสมอ (เช่น admin nav section).
- **ห้ามใช้ `ssr: false`** — มี bug กับ Nuxt 3.x บน Windows (Vite Node IPC socket error).

## Coding Standards
- **Component Pattern:** Use `<script setup lang="ts">`.
- **Server Routes:** Organize logic in `server/api/...` for SQL generation and execution.
- **Naming Conventions:**
  - **Frontend/API:** camelCase (e.g., `isApproved`, `resultCount`).
  - **Database Fields:** Match Vtiger's snake_case (e.g., `account_id`, `productname`).
- **Error Handling:** Implement robust error boundaries for database timeouts or invalid AI-generated SQL.

## UI/UX Guidelines
- **Dashboard:** Show a list of "Pending Approvals" for managers.
- **SQL Editor:** Include a read-only code highlight block to show the generated SQL.
- **Visual Feedback:** Use Loading states during AI generation and SQL execution.
- **Language:**
  - **UI Labels:** Thai language (e.g., "ขออนุมัติดึงข้อมูล", "รายการที่พบ", "สถานะการอนุมัติ").
  - **Code/API:** English.

## Language & Communication
- **Communication Language:** ให้สื่อสาร อธิบายโค้ด และให้คำแนะนำเป็น **ภาษาไทย** ทั้งหมด
- **User Interface (UI):** ใช้ภาษาไทยในส่วนของ Label, Button, และ Message ต่างๆ
- **Comments:** เขียน Comment อธิบาย Logic ที่ซับซ้อนเป็นภาษาไทย
- **Tone:** Professional, Secure, and Technical.

---
> Source: [nathanbc46/aiquery](https://github.com/nathanbc46/aiquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
