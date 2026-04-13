## ergon

> You are Cursor working inside the Ergon Management repository.

You are Cursor working inside the Ergon Management repository.

You must follow these documents as the single source of truth:
- docs/highLevelDesign.md
- docs/lowLevelDesign.md

If there is any conflict, the Low-Level Design wins.

========================
NON-NEGOTIABLE CONSTRAINTS
========================

1) Multi-tenancy and company isolation
- This is a multi-tenant SaaS.
- Every table (except companies) includes company_id.
- Every read/write must be scoped to company_id.
- No cross-company data access is allowed.
- Users belong to exactly one company in v1.

2) RLS is mandatory
- Row Level Security must be enabled on all tenant tables.
- Policies must enforce: row.company_id == current_user.company_id
- Never rely on client-side filtering.
- Never disable RLS for convenience.

3) Supabase Auth only
- Use Supabase Auth for authentication.
- Never store passwords in application tables.
- All auth/session logic must be Supabase-based.

4) Edge Functions are the backend layer
- Use Supabase Edge Functions for backend logic and privileged operations.
- Do NOT build a separate backend server.
- Do NOT add background job infrastructure in v1.
- Edge Functions must:
  - validate JWT
  - extract user id
  - load company_id
  - validate input
  - enforce company scoping
  - return structured JSON responses
  - Never use service role for user-scoped operations unless absolutely required and documented.
- Do not implement business logic in Next.js route handlers.
- Prefer calling Supabase Edge Functions directly from the client using Authorization: Bearer <access_token>.
- If a Next.js route handler is used, it must be a thin proxy that forwards requests to the corresponding Edge Function and returns the response unchanged.

5) Secrets must never reach the client
- Never expose SUPABASE_SERVICE_ROLE_KEY to the browser.
- Never expose OPENAI_API_KEY to the browser.
- Never expose STRIPE_SECRET_KEY or STRIPE_WEBHOOK_SECRET to the browser.
- Only NEXT_PUBLIC_* env vars may exist in browser code.

6) AI is ONLY allowed in the Marketing module
- AI usage is strictly limited to marketing content generation.
- NO AI in jobs, customers, schedule, finance, dashboard, onboarding, billing, or settings.
- Marketing AI must use LangGraph orchestration.
- LangGraph nodes gather/validate/structure inputs.
- Only the GenerateCopy node may call OpenAI.

7) Stripe is isolated to Billing
- All Stripe logic must live in billing endpoints or billing modules.
- Webhook verification is mandatory.
- Never trust client-side payment status.
- Subscription status is stored on companies.subscription_status.

8) Responsive professional UI is required
- Desktop: sidebar visible
- Mobile: sidebar hidden with hamburger menu drawer
- No horizontal scrolling
- Tables collapse to card layouts on small screens
- All authenticated pages must use shared AppShell layout

9) Commenting requirement
- Add lots of comments to core logic.
- Every Edge Function must explain:
  - what it does
  - why it exists
  - inputs/outputs
  - security assumptions
  - company scoping logic
- RLS policies and migrations must be clearly documented with comments.

10) Testing requirement (non-negotiable)

- We write a lot of tests. Testing is mandatory for all critical logic.
- Every new feature must include tests in the same PR.
- Every bug fix must include a regression test.
- A PR that breaks tests cannot be merged.

Critical areas that must be tested:
- Company scoping (no cross-company data leakage)
- RLS-enforced behavior (cross-company attempts must fail)
- Auth checks and session handling
- Job status transitions (lead → scheduled → completed → paid)
- Finance totals (revenue/expense/net) and date range calculations
- Schedule creation and retrieval (jobs + calendar_events)
- Job photo upload flow (signed upload URL + job_photos record creation)
- Marketing generation flow (LangGraph orchestration; OpenAI mocked)
- Stripe webhook processing and subscription_status updates (Stripe mocked)
- Edge Function request validation and response handling

- External services must always be mocked in tests.
  - Never call OpenAI, Stripe, or third-party APIs during test runs.
- Tests must be deterministic and not depend on network access.
- Prefer many small, focused tests over large complex ones.
- If something is difficult to test, the architecture likely needs improvement.

11) Future mobile app compatibility
- v1 is a responsive web app.
- Edge Functions must be client-agnostic so a future mobile app can reuse them.
- Do not embed business rules only in React components.
- Keep shared logic in /lib when appropriate.
- Prefer stable JSON request/response contracts for Edge Functions.
- Avoid browser-only assumptions in core flows.

========================
IMPLEMENTATION GUIDELINES
========================

A) Folder structure conventions
- Keep docs in /docs
- Keep Supabase functions in /supabase/functions
- Keep SQL migrations in /supabase/migrations
- Keep shared UI layout in /app/(app)/layout.tsx or shared AppShell component
- Keep marketing AI logic in a dedicated marketing folder/module
- Keep billing logic in a dedicated billing folder/module

B) Data and API boundaries
- Client UI calls Edge Functions or uses Supabase client under RLS.
- Prefer Edge Functions for workflows and writes.
- Direct Supabase reads are allowed only for simple list/detail queries under RLS.
- Any operation needing elevated privileges must go through Edge Functions.
- Never write direct SQL from client code.

C) Validation and errors
- Validate inputs server-side (Edge Functions).
- Standardize responses:
  - Success: { "data": <payload> }
  - Error: { "error": "message", "code": "ERROR_CODE" }
- UI must show field-level errors when possible.

D) Do NOT invent features
- Do not add new modules outside the MVP.
- Do not add auto-posting, bank sync, payroll, inventory automation, staff roles, or additional AI features in v1.

E) Edge Function contract clarity
- Every Edge Function must clearly document:
  - JSON request schema
  - Query parameters
  - Authentication requirements
  - JSON response schema (success and error)

F) CLIENT DATA ACCESS RULE (UI)
- React components must not import Supabase clients directly.
- Components may only call:
  - hooks in /hooks (client), OR
  - server actions/helpers that call Edge Functions (server), OR
  - simple RLS-protected reads via an approved hook/service.
- All workflows and writes (create/update/delete, onboarding, photo upload signing, finance writes, marketing generate, billing) must go through Supabase Edge Functions.

When unsure, stop and ask for clarification instead of making architectural assumptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ergonmanagement)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ergonmanagement)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
