## gosimy

> **gosimy** is a modern, internationalized webshop for purchasing travel eSIMs. It is built using the Next.js App Router (TypeScript) and leverages Supabase for its backend and authentication.

# GEMINI.md - Project Context

## Project Overview
**gosimy** is a modern, internationalized webshop for purchasing travel eSIMs. It is built using the Next.js App Router (TypeScript) and leverages Supabase for its backend and authentication.

- **Primary Technologies:** Next.js 15+, React 19, TypeScript, Tailwind CSS, Radix UI.
- **Backend & Database:** Supabase (PostgreSQL, Auth, SSR).
- **Internationalization:** `next-intl` (Supported: English, German, French, Spanish).
- **Payments:** Dodo Payments.
- **Email:** Resend & React Email.
- **Monitoring:** Sentry.
- **State Management:** Zustand.
- **Validation:** Zod.

## Architecture & Directory Structure
- `app/[locale]/`: The localized frontend pages (auth, shop, checkout, profile, etc.).
- `app/admin/`: Admin dashboard for managing orders, refunds, and monitoring.
- `app/api/`: API routes for eSIM provisioning, checkout, and webhooks.
- `lib/`: Domain-specific business logic:
    - `fulfillment/`: Core order fulfillment and eSIM provisioning logic.
    - `providers/`: Integration with eSIM provider APIs (e.g., eSIM Access).
    - `supabase/`: Supabase client configurations (server, client, admin).
    - `admin/`: Admin alerts and audit logging.
- `components/`: UI components organized by domain (cart, shop, portal, admin, etc.).
- `messages/`: Translation JSON files for `next-intl`.
- `supabase/`: Database migrations and local setup.
- `tests/`: E2E tests (Playwright) and unit tests (Vitest).

## Building and Running
- **Development:** `npm run dev` (uses webpack for Next.js 15).
- **Build:** `npm run build`.
- **Start:** `npm run start`.
- **Linting:** `npm run lint`.
- **Unit/Integration Tests:** `npm run test` (Vitest).
- **E2E Tests:** `npm run test:e2e` (Playwright).
- **Coverage:** `npm run test:coverage`.

## Development Conventions
- **Surgical Updates:** When modifying existing logic (especially in `lib/fulfillment`), ensure you maintain atomicity and handle errors gracefully.
- **Type Safety:** Heavily utilize Zod for validation and ensure TypeScript types are strictly followed (especially with `database.types.ts`).
- **Security:** CSP and strict security headers are configured in `next.config.ts`. Always respect these when adding new external scripts or styles.
- **i18n:** All user-facing strings must be added to `messages/*.json` and retrieved using `useTranslations` (client) or `getTranslations` (server).
- **Supabase Clients:**
    - Use `createSupabaseServerClient()` for server-side logic (RSC, Actions).
    - Use `createSupabaseAdminClient()` in `lib/` for tasks requiring service-role privileges (fulfillment, admin alerts).
- **Fulfillment:** Order fulfillment involves polling eSIM providers. Ensure any changes to the polling logic (`POLL_DELAYS_MS`) consider provider rate limits and response times.

## Key Scripts
- `scripts/seed-packages.js`: Populates the `packages_cache` table from eSIM providers.
- `scripts/seed-admin.ts`: Setup initial admin users.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MrMomo2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MrMomo2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
