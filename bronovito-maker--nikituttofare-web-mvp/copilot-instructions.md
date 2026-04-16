## nikituttofare-web-mvp

> - **Frontend:** Next.js 15 (App Router), React Server Components, TypeScript.

# CURSOR SYSTEM RULES

## 1. TECH STACK (STRICT)
- **Frontend:** Next.js 15 (App Router), React Server Components, TypeScript.
- **Styling:** Tailwind CSS, Shadcn/UI (Radix Primitives), Lucide React Icons.
- **Backend/DB:** Supabase (PostgreSQL, Auth, Storage, Realtime).
- **AI Engine:** Google Gemini API (`gemini-1.5-flash` for chat, `gemini-1.5-pro` for reasoning).
- **Notification:** Telegram Bot API (via `node-telegram-bot-api` or simple fetch).
- **State:** React Query (TanStack Query) for server state, URL search params for UI state.

## 2. CODING STANDARDS
- **TypeScript:** USE STRICT TYPES. No `any`. Define interfaces in `@/types`.
- **Components:** Functional components only. Use `const` definitions.
- **Server Actions:** Prefer Server Actions over API Routes for form submissions.
- **Error Handling:** Always wrap async calls in try/catch. Use `sonner` for UI feedback.
- **Imports:** Use absolute imports `@/components/...`.
- **Formatting:** Prettier standard.

## 3. BEHAVIORAL RULES (NO HALLUCINATIONS)
- **NO PLACEHOLDERS:** Never leave comments like `// insert logic here`. Write the actual code.
- **NO MOCK DATA:** If database is ready, use real Supabase calls.
- **CHECK EXISTING:** Before creating a file, check if it exists to avoid duplicates.
- **ZERO COGNITIVE LOAD:** UI must be extremely simple. Big buttons, clear texts, no clutter.

## 4. PROJECT CONTEXT
This is "NikiTuttoFare", an emergency service app for Home/HORECA maintenance.
Users are often stressed or non-tech savvy. The interface must be reassuring and fast.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bronovito-maker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
