## video-loan-origination-system

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Context

## Discoveries
- **Prisma 7** requires a driver adapter (not `url = env()` in schema). Used `@prisma/adapter-libsql` with `@libsql/client` for SQLite. The generated client goes to `app/generated/prisma/`.
- **Prisma client singleton** needs `dotenv/config` imported before instantiating, and the adapter (libsql) must also be stored globally.
- **Session lookup bug**: The `/apply` route was ignoring phone hash when checking for existing sessions, causing different phones with the same campaign to always resume the same session. Fixed by adding `phoneHash` to session metadata and filtering with `contains: phoneHash`.
- **Rate limiter lives in `/apply` route** (not `/api/sessions`) — it records every request, blocking the 4th+ session start per phone.
- **Prisma cannot be imported in Client Components** — caused a build error. Solved by moving `STEPS`/`STEP_LABELS`/`getNextStep` to `lib/steps.ts` (no Prisma/event-logger imports), keeping `session-steps.ts` for server-side only logic.
- **`fast-xml-parser` v5** exports `XMLParser` (not `load`). The parser is used in `lib/services/aadhaar-parser.ts`.
- **Jest + Prisma ESM**: Jest couldn't parse `import.meta.url` in the generated Prisma client. Fixed by creating `__tests__/__mocks__/prisma.ts` and `event-logger.ts`, and mapping `@/lib/prisma` and `@/lib/event-logger` in `jest.config.ts`. Module mapper entries must be ordered before the catch-all `@/(.*)$` pattern.
- **ESLint `no-require-imports`**: Used `require("crypto")` for `createHash` — replaced with top-level ESM `import { createHash } from "crypto"`.
- **Event handlers cannot be passed from Server to Client Components** — removed `onSubmit` prop from `EligibilityForm`, making it self-contained with internal API calls.
- **Next.js 16 `params`**: Route params are now `Promise<{ sessionId: string }>`, must be `await`ed in server components.
- **`base64url` share code**: The share code uses `base64url` encoding which includes `-` and `_`. Fixed validation regex from `^[A-Za-z0-9]{8}$` to `^[A-Za-z0-9_-]{8}$`.

## Accomplished

### Phase 1: Project Infrastructure
- Prisma 7 + libSQL + SQLite schema (13 tables)
- All adapter interfaces (`VCIP`, `LLM`, `Bureau`, `OCR`, `STT`, SMS/Email) with mock implementations
- HMAC-SHA256 campaign link signing (`lib/campaign.ts`)
- Admin auth with bcrypt + JWT + roles (`lib/admin-auth.ts`)
- Event logging service (`lib/event-logger.ts`)
- All route scaffolds (borrower + admin API routes)
- `prisma/seed.ts` — creates admin user, default config, default campaign
- `__tests__/campaign.test.ts` — 13 unit tests

### Phase 2: Campaign Link & Session Management
- Full `/apply` route lifecycle: verify → rate-limit → resume/create → notify → redirect
- Rate limiter (`lib/services/rate-limiter.ts`) — in-memory, 3 per phone per hour
- Step service (`lib/services/session-steps.ts` + `lib/steps.ts`)
- Video retry service (`lib/services/video-retry.ts`)
- Notification service (`lib/services/notifications.ts`)
- UI: `ProgressTracker`, `RetryBanner`, `EligibilityForm` client components
- Stub pages for Video, Offer, Disbursement
- `EligibilityForm` — self-contained client component with API calls
- `EligibilityAPI` (`/api/risk/eligibility`) — config-driven eligibility rules
- Video API (`/api/video/[sessionId]`) — retry, start actions
- `__tests__/session-steps.test.ts`, `__tests__/rate-limiter.test.ts`, `__tests__/eligibility.test.ts`
- All 8 Phase 2 ACs verified end-to-end via curl + SQLite queries

### Phase 3: KYC Document Collection — COMPLETE
- `lib/encryption.ts` — AES-256-GCM file encryption
- `lib/file-storage.ts` — encrypted file store with session-scoped directories
- `lib/services/aadhaar-parser.ts` — XML parsing + share code validation (SHA-256 based)
- `lib/services/kyc-verification.ts` — document upload processing + mismatch detection
- `app/api/kyc/[sessionId]/route.ts` — multipart upload, XML parsing, OCR, mismatch detection
- `app/api/kyc/[sessionId]/documents/route.ts` — GET docs, POST advance
- `app/components/AadhaarUploadForm.tsx` — drag-and-drop XML upload, share code input
- `app/components/PANUploadForm.tsx` — PAN number input, optional card image upload
- `app/components/KycForm.tsx` — tabbed interface, auto-fill display, mismatch error banner
- Updated `app/apply/[sessionId]/kyc/page.tsx` to use new `KycForm` component
- `__tests__/aadhaar-parser.test.ts` — 7 unit tests
- All ACs verified end-to-end via curl: Aadhaar XML upload, PAN upload, session advance

### Phase 4: Video Call Shell — COMPLETE
- `lib/agora.ts` — Agora adapter interface with mock implementation (IAgoraProvider)
- `app/components/VideoCall.tsx` — client component with camera permission, live video, guidance prompts, mute/video toggle
- `app/api/video/[sessionId]/route.ts` — full CRUD for video sessions (start, end, retry, flag, metadata, get config)
- `app/apply/[sessionId]/video/page.tsx` — updated to use VideoCall component
- Guidance sequence: intro → face position → Aadhaar front → Aadhaar back → PAN → confirmation
- Geo-location capture with GPS (IP fallback) after explicit consent
- Video quality monitoring with warnings for poor connection
- Retry logic: max 3 retries, session flagged on max exceeded
- All video events logged to EventLogs
- `__tests__/video-retry.test.ts` — 12 unit tests

## Next: Phase 5 — STT & Consent Capture

---
> Source: [Rachana901070/video-loan-origination-system](https://github.com/Rachana901070/video-loan-origination-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
