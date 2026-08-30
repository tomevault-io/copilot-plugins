## forgefly

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm test         # Run tests (watch mode)
pnpm test:ci      # Run tests with coverage (single run)
pnpm test:watch   # Run tests in watch mode
pnpm test:ui      # Run tests with Vitest UI
pnpm lint         # Full lint: type-check + biome + tailwind CSS check
pnpm type-check   # TypeScript type checking only
pnpm format       # Format with Prettier
```

To run a single test file:
```bash
pnpm vitest run src/test/stripe.test.ts
```

Supabase CLI (project is **Forgefly-prod** — there is no separate local/staging project; always confirm with the user before pushing migrations or deploying functions):
```bash
supabase migration list          # compare local migrations against the remote
supabase db push                 # apply pending migrations to Forgefly-prod
supabase functions deploy <name> # deploy a single edge function
```

## Environment

Requires a `.env` file with:
```
VITE_SUPABASE_URL=...
VITE_SUPABASE_ANON_KEY=...
```

Supabase Edge Functions require secrets set server-side (Supabase dashboard → Edge Functions → Secrets), grouped by what they power:
- **Core**: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` / `SUPABASE_SERVICE_KEY`, `SITE_URL`
- **AI**: `ANTHROPIC_API_KEY` (Freeda, promotion captions), `OPENAI_API_KEY` (gpt-image-2 promotion images), `PERPLEXITY_API_KEY` (Sonar search-grounded research, market research pipeline)
- **Payments**: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_AGENCY_MONTHLY_PRICE_ID`, `STRIPE_AGENCY_YEARLY_PRICE_ID`
- **Social**: `META_APP_ID` / `META_APP_SECRET` (Instagram + Facebook + WhatsApp OAuth and publishing, one Meta app for all three), `INSTAGRAM_APP_ID` / `INSTAGRAM_APP_SECRET`, `WHATSAPP_WEBHOOK_VERIFY_TOKEN`
- **Promotions video**: `SHOTSTACK_API_KEY` (currently pinned to Shotstack's free/watermarked sandbox endpoint — swap to a production key + endpoint before relying on unwatermarked Reels)
- **Misc**: `RESEND_API_KEY` (email), `REVIEW_JWT_SECRET` (review-link tokens), `APPLE_TEAM_ID` / `APPLE_PASS_TYPE_ID` (wallet pass generation)

## Architecture

**Forgefly** is an AI-powered business OS for freelancers/solopreneurs. React 18 + TypeScript SPA with Supabase as the backend, Stripe for payments, and Meta/Shotstack integrations for social promotion.

### Auth & routing

- `AuthContext` (`src/contexts/AuthContext.tsx`) manages Supabase session, user `Profile`, and `Subscription` (freelancer vs. agency tier). Authentication uses a `username@miaoda.com` email convention — users sign in with a username, not an email.
- `RouteGuard` (`src/components/common/RouteGuard.tsx`) redirects unauthenticated users to `/login`. Public routes are declared in `routes.tsx` via `public: true`.
- `App.tsx` splits routes into two groups: public routes render without a layout wrapper; protected routes render inside `MainLayout` (Sidebar + Outlet + AICopilot).
- Notable public routes beyond auth: `/preview` (pre-signup AI-generated business preview, `GeneratedPortalPage.tsx` — a deliberately standalone read-only clone, not a reuse of the real dashboard; see `src/components/preview/` if touching this), `/p/:slug` (a freelancer's public portfolio), `/portal/:token` (client portal), `/documentation` (in-app help docs), `/contact`.

### Data layer

All DB access goes through `src/services/` — one file per domain: `clientService.ts`, `projectService.ts`, `invoiceService.ts`, `proposalService.ts`, `paymentService.ts`, `packageService.ts`, `calendarService.ts`, `financeService.ts`, `timeService.ts`, `portalFileService.ts`, `dashboardService.ts` (cross-cutting reads for the Dashboard and Freeda's KPI catalog — single source of truth so numbers can't drift between the two), `socialService.ts` (platform connections, competitor intel), `promotionService.ts` (AI-generated promotion drafts and publishing). Services call Supabase directly via `src/db/supabase.ts`. Several services expose a `subscribeTo*` function using Supabase Realtime for live updates.

All domain types are in `src/types/types.ts` — `Client`, `Project`, `Invoice`, `Proposal`, `Payment`, `Package`, `CalendarEvent`, `Automation`, `Profile`, `BusinessProfile`, plus social/promotion types (`SocialConnectionStatus`, `Promotion`, `SocialPostTargetPlatform`).

### Supabase Edge Functions

Located in `supabase/functions/`, written in Deno/TypeScript. Grouped by area (see each function's `index.ts` for detail):

| Area | Functions |
|---|---|
| **Freeda / AI** | `ai-gateway` (sole LLM entry point — see Freeda section below), `extract-receipt`, `generate-visibility-kit`, `research-company`, `research-competitor`, `quarterly-review-insight`, `trigger-nudges`, `generate-market-research` (Perplexity Sonar research + Claude synthesis, see below) |
| **Social & Promotions** | `social-oauth-callback`, `social-facebook-select-page`, `social-disconnect`, `get-social-status`, `social-publish-instagram`, `social-publish-facebook`, `social-send-whatsapp`, `send-whatsapp-message`, `whatsapp-webhook`, `handle-reply-intent`, `generate-promotion`, `check-video-render` |
| **Payments/billing** | `create-checkout-session`, `create-invoice-checkout`, `create-subscription-checkout`, `portal-create-checkout`, `verify-stripe-payment`, `stripe-webhook`, `subscription-webhook`, `create-connect-account`, `get-connect-status` |
| **Client portal** | `generate-portal-link`, `portal-approve-proposal`, `upload-portal-file`, `notify-portal-file-shared`, `select-portal-testimonials` |
| **Reviews / outreach** | `submit-review`, `send-review-requests`, `schedule-review-request`, `submit-proposal-request` |
| **Onboarding** | `mark-milestone` |
| **Misc** | `send-email`, `send-daily-digest`, `generate-wallet-pass`, `connect-toggl`, `sync-toggl-entries`, `request-deletion-otp`, `confirm-account-deletion` |

`ai-copilot` (a legacy OpenAI-based function) has been deleted — Freeda has always been Claude-powered via `ai-gateway`.

### Freeda (AI Copilot)

The `AICopilot` component (`src/components/layouts/AICopilot.tsx`) is the single Freeda surface — a resizable right-side panel, opened via the "Ask Freeda" button in `AppSidebar`. There used to be a second surface (a standalone "Upgrade my Business" command bar, `CommandBar.tsx`) for business-data updates; it was retired and merged into this panel so users only have one place to type anything — an update, a question, or a request to take an action.

It calls `ai-gateway`'s `mode: 'freeda'`, which routes the message and returns a `kind`-tagged response the panel renders differently per bucket, all inline within the same scrolling chat history:
- `update` → a diff card (`buildDiffLines`/`applyBusinessDiff` in `src/lib/businessDiff.ts`) with Dismiss/Apply — applying writes `extracted_data` and syncs new services/contacts into their own tables.
- `query` → either a live stat/list card sourced from `src/config/freedaKpiCatalog.ts` + `src/services/dashboardService.ts` (same data the Dashboard renders, so the numbers can't drift), or a freeform grounded answer if the question doesn't match a known KPI.
- `action` → a proposal card (recipients + an editable drafted message, with a picker if recipients are ambiguous). Nothing sends automatically from the classified intent — the user edits the draft and clicks an explicit "Send to N clients" button, which calls `handleActionExecute` in `ai-gateway` as a separate request from the proposal step. See the security boundary section below for why propose/execute stay two distinct steps.
- `support` / `off_topic` → a plain chat reply, rendered as markdown (`react-markdown`).

### AI-to-database security boundary — read before adding any AI-driven DB read/write/action

This applies to every current and future `ai-gateway` mode (`extract`, `chat`, and any query/action capability added later) and to any other code path where LLM output can influence a database read, write, or side-effecting action (sending a message, marking something paid, bulk operations, etc.).

**The LLM's output must only ever be treated as data (values to fill in), never as query logic (which table, which row, which operation).** Concretely:
- The owning `user_id` / `business_id` on any write must come from the authenticated session (e.g. `business.user_id` from `useBusiness()`), never from a field the model returned — even if the model's JSON contains a `user_id`-shaped field, application code must ignore it and hardcode the real one. See `applyBusinessDiff()` in `src/lib/businessDiff.ts` for the reference pattern.
- Every table touched by AI-influenced writes must have RLS scoped to `auth.uid()` (see `businesses`, `services`, `clients` policies) — this is the actual enforcement boundary, not app-code discipline alone.
- The model must never be allowed to construct or select raw SQL, table names, or target rows. `query` intent already follows this: `matchKpiCatalog()` in `ai-gateway` only ever returns a *catalog id*, never a computed value — the frontend fetches the live number itself via the same code the Dashboard uses. Any future open-ended query capability (beyond the fixed KPI catalog) needs the same shape: a fixed allow-list of pre-defined, parameterized functions the model can invoke, never raw SQL construction.
- Any action with a real-world side effect (sending a message/email, bulk operations, marking something paid) must render a review/confirm step before executing — never fire directly from a classified intent. `action` intent follows this as two separate server calls: `handleActionPropose` returns only a recipient list + drafted message (nothing sent yet); `handleActionExecute` runs only after the user has edited/confirmed the draft in the UI and clicked Send. Any new AI-driven side effect needs the same two-call shape — never collapse propose and execute into one request.
- This applies just as much to `generate-promotion` (AI writes a caption + generates an image) — the model's output is copy and pixels, never a target platform, business id, or publish decision. Publishing always requires the separate, explicit `social-publish-*` step reviewed on the Draft tab, never an automatic fire from generation.

This is what keeps prompt injection from becoming a database compromise. It does not come for free when adding new AI-driven capabilities — it has to be deliberately preserved each time.

### Application security — read `sec-rev/APPSEC.md` for anything sensitive

The AI-to-database boundary above is one piece of a broader application-security reference at **`sec-rev/APPSEC.md`** — prompt injection from third parties, webhook signature verification, RLS patterns (including the "anyone with a token can read row X" pattern that RLS's `using (true)` cannot safely express), edge-function caller-identity checks, SSRF guarding for any external fetch, file upload validation, and OTP/verification-code hardening. Six auto-firing skills back it (`.claude/skills/appsec-*`) that trigger when their area is touched, plus `.claude/skills/security-review` — a re-runnable full-codebase security review loop, distinct from `/security-review`'s diff-only scope. Past review cycles are logged in `sec-rev/findings/`.

### Social & Promotions

`SocialPage.tsx` (`/dashboard/social`, agency-tier gated) is the workspace for connecting social accounts and running AI-drafted promotions. See `src/components/promotions/` for the component breakdown if working on this area.

- **Connect**: Instagram, Facebook, and WhatsApp all OAuth through the same Meta app (`META_APP_ID`/`META_APP_SECRET`). `social-oauth-callback` does the token exchange and upserts `social_connections` — a service-role-only table with no client RLS, so platform tokens never reach the browser. Facebook accounts with multiple Pages route through an extra one-time picker (`social-facebook-select-page`); unselected Page tokens are stashed server-side only.
- **Generate**: `generate-promotion` uses Claude Haiku for the caption and OpenAI's gpt-image-2 for a text-free illustrative image (baked-in AI text renders unreliably), and — for the `featured_openai` path only — also submits a short Ken Burns pan/zoom video render to Shotstack for an Instagram/Facebook Reel; `check-video-render` polls that render's status.
- **Review & publish**: drafts live on the Draft tab (`DraftPromotionCard.tsx` → `EditPromotionModal.tsx` → `PublishWorkflowModal.tsx`), which publishes photo then Reel sequentially per platform via `social-publish-instagram` / `social-publish-facebook`, each leg failing independently. `promotionService.ts` (`LIVE_PLATFORMS = ['instagram', 'facebook']`) and `socialService.ts` (connection status, competitor intel) back this end to end.
- WhatsApp is used both for the social module (broadcast via `social-send-whatsapp`) and separately for transactional sends like invoice payment links (`send-whatsapp-message`); inbound replies land via `whatsapp-webhook` (Meta-called, `--no-verify-jwt`) and `handle-reply-intent`.

### Market Research (in progress — schema, edge function, and trigger exist, no review UI yet)

`generate-market-research` produces a per-business landscape report: Perplexity's Sonar API runs three fixed, never-model-chosen research angles (local competitors, referral-partner leads, discovery channels) in parallel, then a single Claude Sonnet call synthesizes the results into a `market_summary` plus a fixed allow-list of item types (`outreach_draft`, `channel_signup_suggestion`, `pricing_note`, `positioning_insight`), each tagged `actionable` or `fyi` — the tag is derived server-side from `item_type`, never trusted from the model. Actionable items get a real outreach message drafted by a parallel per-item Claude Haiku fan-out (one call each, so one failure never blocks the rest); fyi items use the synthesis step's insight text directly. Perplexity's returned research is third-party web content and is explicitly framed as untrusted reference data in the synthesis prompt, never as instructions.

Storage: `market_research` (one job/status row per business — `pending`/`running`/`ready`/`failed`, holds `market_summary` + `citations`) and `market_research_items` (the individual items, `status` lifecycle `new` → `approved`/`rejected`/`dismissed` → `sent`). RLS is select-only on `market_research` and select+update on `market_research_items` for authenticated users — both tables are only ever written by the edge function's service-role client. Currently runs once per business (no re-run yet); a `trigger_source` column (`generate_call` | `manual`) already anticipates a future manual re-run path, which should also add a cooldown before it ships.

Triggered fire-and-forget (not awaited, never blocks onboarding, failure is non-fatal) right after a *new* business row is created — both onboarding paths call it: `AuthCallbackPage.tsx`'s `saveBusiness()` (email-verification redirect flow) and `useCurrentBusiness.ts`'s `fetchBusiness()` insert branch (already-logged-in flow). Deliberately not called from the `existing?.id` update branch in either file — this only ever fires once, for a genuinely new business.

**Not yet built**: the swipe-approve queue UI and the Market Research tab. Approving an actionable item is scoped to only ever flip its `status` — the real send must still go through Freeda's existing propose/execute two-phase confirm (see the AI-to-database security boundary above), never fire directly from an approval.

### Onboarding & Getting Started

`businesses.onboarding_milestones` (jsonb) tracks six flags: `business_created` (auto-set silently on first load), `services_reviewed`, `prospect_added`, `proposal_sent`, `portfolio_shared`, `social_connected`. Each is set once via the `mark-milestone` edge function (JWT-verified, allow-listed milestone names, logs every completion/skip to the append-only `onboarding_events` table) from its natural trigger point in the app — `PackagesPage.tsx`, `PipelinePage.tsx`, `ProposalsPage.tsx`, `BrandKitPage.tsx`, `SocialPage.tsx` respectively.

`GettingStartedChecklist.tsx` (`src/components/common/`) renders all five user-facing milestones as a persistent checklist on the Dashboard, with a progress bar and a durable dismiss (`businesses.getting_started_dismissed`, a plain preference column — not a milestone, not logged to `onboarding_events`). The card auto-hides once all five are done; dismissing collapses it to a small resumable progress-ring pill rather than removing it outright.

### Subscription tiers

`isAgency` in `AuthContext` is `true` only when `subscription.tier === 'agency' && subscription.status === 'active'`. **Agency-tier gating is currently disabled repo-wide (temporary — "for now").** It used to gate Social/Promotions (`SocialPage.tsx`), the team-member invite on `ClientsPage.tsx`, and the AI-promotions-ready banner on `DashboardPage.tsx` — those `isAgency` checks were removed so every feature is available to every user regardless of tier. `isAgency` itself is left intact and unused in `AuthContext` so re-enabling gating later is a small diff (add the checks back), not a rebuild. Stripe Checkout for the upgrade itself still runs through `create-subscription-checkout` / `subscription-webhook`; Stripe Connect (`create-connect-account`, `get-connect-status`) is separate — it's for routing client payments to the freelancer, not for the Forgefly subscription itself.

### UI

- Components in `src/components/ui/` are shadcn/ui primitives — treat them as library code.
- `src/components/common/` contains app-specific shared components; `src/components/promotions/` is the Social/Promotions component set; `src/components/preview/` backs the pre-signup `/preview` page only — do not import these into real dashboard pages or vice versa, they're intentionally separate (see Auth & routing above).
- `src/lib/utils.ts` exports `cn()` (clsx + tailwind-merge) used everywhere for conditional class merging.
- The `@` path alias resolves to `src/`.

### Documentation

`src/pages/DocumentationPage.tsx` (`/documentation`, public, linked from the site footer and the contact page) is the in-app help center — a flat list of `DocSection`/`DocSubheading` blocks. It currently only covers Instagram and WhatsApp; add new `DocSection` blocks here (don't build new page structure) as features like Freeda, Facebook, promotions, or the Getting Started checklist need end-user docs.

### Linting rules

Biome (`biome.json`) enforces: no undeclared dependencies, no redeclare, no CommonJS (`require`). Formatter is disabled in Biome; Prettier handles formatting. The `lint` script also validates Tailwind CSS output and runs a build smoke test via `.rules/testBuild.sh`.

---
> Source: [souravpn/forgefly](https://github.com/souravpn/forgefly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
