## ai-resume-platform

> Full copy in `RESEARCH_PROTOCOL.md`. Goal = minimize error + make every claim auditable; 100% accuracy is impossible so **abstain when unsure**. Rigor scales to stakes.

# Memory — AI Resume Studio

## Research Protocol (v3 — apply whenever Akil asks you to research a topic)
Full copy in `RESEARCH_PROTOCOL.md`. Goal = minimize error + make every claim auditable; 100% accuracy is impossible so **abstain when unsure**. Rigor scales to stakes.
0. **Read the codebase first** when the research touches our product — ground in what exists before searching.
1. **Never fabricate** a URL, quote, stat, date, author, or study. If you didn't fetch it this session, you don't have it.
2. **Fetch real sources; never quote a search-engine summary.** Open the actual page for any load-bearing claim; label snippet-only claims "SNIPPET — UNVERIFIED."
3. **Read targeted, not everything** — extract the relevant passage; don't dump whole long docs (noise + lost-in-the-middle lower accuracy).
4. **Failed fetch** (JS shell/paywall/blocked) → use browser tools or say you couldn't read it; never quote a shell or guess.
5. **Quote honestly** — "verbatim" only if copied from a page you opened; else "paraphrase"/"snippet."
6. **Independence > quantity** — trace to primary origin; 2 independent min, 3+ for numbers/high-stakes; don't count circular/SEO echoes; stop at saturation.
7. **Numbers/dates/volatile facts** → 2+ independent fetched sources + "as of [date]"; flag >~2yr or fast-moving-with-old-sources.
8. **Disconfirming search — and act on it**; if it weakens the claim the headline must say so; present disagreement as a range.
9. **Abstain when unsure** — "couldn't verify" is preferred over a guess; never assert an unverified claim as fact, even softened.
10. **Confidence by evidence** (High = ≥2 independent primary opened; Medium = 1 primary / 2 secondary; Low = single/vendor/undated/snippet). Cap vendor/SEO/AI-content.
11. **Separate sourced from inferred** — label your own analysis; show queries run + pages opened for auditability.

## Me
Akil Harrington, founder of AI Resume Studio. Non-technical. Building an AI-powered resume optimization SaaS.

## Project
| Field | Value |
|-------|-------|
| **Name** | AI Resume Studio |
| **Path** | `/Users/akilharrington/ai-resume-platform` |
| **Backend** | FastAPI (Python), runs on port 8000 |
| **Frontend** | React + TypeScript + Vite, runs on port 5173 |
| **AI** | Anthropic Claude claude-sonnet-4-6 (all features) |
| **Auth** | Supabase — fully wired (login, signup, session, profiles table) |
| **Payments** | Stripe — checkout live, webhook wired, price IDs set |
| **Database** | Supabase — `profiles` table with `is_pro`, `stripe_customer_id`, `stripe_subscription_id` |
| **Test resume** | Danielle Richards (Senior Operations Coordinator) |

## Stack Terms
| Term | Meaning |
|------|---------|
| **semantic scorer** | Claude-powered ATS scorer (`semantic_ats_service.py`) — primary |
| **rule-based scorer** | Keyword-matching ATS scorer (`ats_service.py`) — fallback / keyword extraction |
| **FORCE_PRO** | `backend/.env` flag — set to `true` to bypass pro gate locally without Stripe |
| **pro gate** | UpgradePrompt shown on Optimize/Cover Letter/LinkedIn tabs when `isPro=false` |
| **JD** | Job description (pasted by user for ATS scan) |
| **semantic scan** | ATS scan using Claude — requires API credits |

## Current State (after session 19 — production-readiness audit + P0 code fixes)

### ✅ Done
- Claude semantic ATS scorer — 6 dimensions (Human Readability 5%, Keyword Alignment 30%)
- Optimizer prompt: banned 20 AI buzzwords, preserves voice, demands specificity and quantification
- Cover letter prompt: banned hollow openers, enforces human-sounding specific output
- PDF download: @react-pdf/renderer wired into OptimizeTab — 3 templates (Professional, Modern, Executive)
- Pro gate: real enforcement via `isPro` from `/api/user/pro-status`; `FORCE_PRO` env override
- File size limit: 5MB on upload endpoint; PDF magic bytes check (`%PDF-` header)
- Axios timeout: 60s with readable error message
- Scorer consistency: optimize uses semantic scorer for displayed before/after scores
- Full project cleanup: zero dead files, no duplicate frontends (session 13: removed DashboardTab, Sidebar, SkeletonLoader, Button, scoreUtils + dead build_resume_optimization_prompt/optimize_resume_text from resume_service.py)
- Company vision document: `AI-Resume-Studio-Vision.docx`
- **Supabase auth**: signup, login, logout, session persistence via `AuthContext`
- **Supabase profiles table**: auto-created on signup, `is_pro` field, RLS enabled
- **Stripe checkout**: live with real price IDs (monthly $19, one-time $49)
- **Stripe webhook**: `/api/payments/webhook` flips `is_pro=true` in Supabase on `checkout.session.completed`
- **Auth routing**: unauthenticated users redirected to `/login`; landing page routes to `/signup` or `/workspace` based on session
- **Login/Signup pages**: `/login` and `/signup` with email confirmation flow
- **Workspace header**: shows user email, PRO badge, Sign Out button, dark mode toggle
- **Homebrew + Stripe CLI**: installed and authenticated locally
- **Supabase + Stripe keys**: all wired into both `.env` files
- **Security hardening**:
  - JWT verification on every backend endpoint via `get_current_user()` FastAPI Depends
  - Server-side pro gate via `require_pro()` — 403 before any Claude call; Supabase outage → 503 (not silent demotion)
  - Rate limiting via slowapi: 20/min upload, 10/min scan, 5/min optimize/cover/linkedin, 30/min pro-status
  - FORCE_PRO production guard — `sys.exit(1)` if ENVIRONMENT=production + FORCE_PRO=true
  - Subscription cancellation webhook (`customer.subscription.deleted` → flips `is_pro=false`) — logs on failure, no silent pass
  - HTTP security headers middleware: X-Content-Type-Options, X-Frame-Options, Referrer-Policy, HSTS (prod only)
  - Startup env var validation (production) — `sys.exit(1)` on missing critical vars
- **Claude fallback error handling**: `AIUnavailableError` propagates auth/rate/connection errors as 503 with readable messages
- **React error boundaries**: `ErrorBoundary` class component wraps each tab; shows named error card + "Try again" button
- **WorkspacePage split**: monolith broken into feature files — 0 TypeScript errors
  - `src/features/workspace/shared.tsx` — LoadingCard, EmptyState, EmptyCard, UpgradePrompt
  - `src/features/workspace/ScanTab.tsx` — ATS scan results, pro-gated recruiter verdict + strengths/gaps
  - `src/features/workspace/OptimizeTab.tsx` — resume optimizer, before/after scores, PDF download
  - `src/features/workspace/CoverLetterTab.tsx` — cover letter generator
  - `src/features/workspace/LinkedInTab.tsx` — LinkedIn headline + About optimizer
  - `src/pages/WorkspacePage.tsx` — **guided 5-step flow** (see session 12 below)
- **Principal code review — all 28 actionable issues resolved**:
  - CRITICAL: sync def handlers (thread pool, no event loop blocking), fabricated +3 removed, /health checks real deps
  - HIGH: singleton clients, silent excepts removed, Router.navigate, max_tokens 8192, rate limit pro-status, structured logging, prompt injection XML delimiters, startup env validation, 26-test pytest suite
  - MEDIUM: React hooks ordering, enabled:!!user guard, LCS dep array fixed, SSE null guard, scaleX animation, Haiku model alias, security headers
  - LOW: `--success` color → #047857 (WCAG AA 4.54:1), high-visibility focus rings (button/a/[role=tab])
  - Deferred: API versioning prefix (pre-launch disruption), Zod runtime validation (half-day project)
- **Privacy policy + Terms of Service**: `/privacy` and `/terms` routes, linked in footer and signup
- **Backend PDF service** (`backend/services/pdf_service.py`): ReportLab renders all 3 resume templates + cover letter; endpoints `/api/resume/download-pdf` and `/api/cover-letter/download-pdf`
- **Dark mode — semantic token architecture (GitHub palette)**:
  - `ThemeContext.tsx` — React context, `localStorage` persistence, `prefers-color-scheme` OS fallback
  - `globals.css` — primitive tokens never change; semantic role tokens (`--text-primary`, `--surface-0`, etc.) override per theme
  - GitHub dark palette: page #0D1117, cards #161B22, raised #21262D, overlay #2D333B (tonal elevation)
  - Toggle button in both workspace header and landing page nav
  - All pages updated: LandingPage, WorkspacePage, LoginPage, SignupPage, PricingPage, PrivacyPage, TermsPage, all workspace tabs
  - TypeScript clean (0 errors) after full migration

- **Production deployment (session 16) — fully live on Render + Vercel**:
  - **Backend**: Render (auto-deploy from `main` branch) — `https://ai-resume-studio-api.onrender.com`
  - **Frontend**: Vercel — `https://ai-resume-platform-hazel.vercel.app`
  - **PostgREST 401 fix**: Removed hand-rolled HS256 JWT in `supabase_service.py`; now uses real `sb_secret_*` key on `apikey` header only (Supabase gateway translates internally). Added user-JWT + anon-key path for pro-status reads (most reliable path)
  - **CORS fix**: `allow_origin_regex=r"https://ai-resume-platform[a-zA-Z0-9\-]*\.vercel\.app"` in CORSMiddleware — covers all Vercel preview URLs
  - **CoverLetterTab env var fix**: `VITE_API_BASE_URL` → `VITE_API_URL` (was silently falling back to localhost in production for PDF download)
  - **Stripe webhook fix**: Production `STRIPE_WEBHOOK_SECRET` on Render was the local CLI test secret — updated to production signing secret from Stripe dashboard; all events now 200 OK
  - **Email confirmation URL**: Supabase Site URL set to `https://ai-resume-platform-hazel.vercel.app` (was localhost)
  - **Supabase key format**: New format uses `sb_publishable_*` (anon) and `sb_secret_*` (service role) — not JWTs
  - **Render env vars set**: `ENVIRONMENT=production`, `FRONTEND_URL`, `SUPABASE_ANON_KEY`, `ALLOWED_ORIGINS`, `STRIPE_WEBHOOK_SECRET` (production)
  - **Vercel env vars set**: `VITE_API_URL`, `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `VITE_STRIPE_PUBLISHABLE_KEY`
  - **Supabase RLS + grants fix (session 17)**: Legacy JWT keys were disabled by Supabase on 2026-06-15. Switched to new `sb_publishable_*` (anon) + user JWT path for all reads. Fixed missing table-level grants — ran SQL to `GRANT SELECT, INSERT, UPDATE ON public.profiles TO authenticated, anon` and `GRANT ALL TO service_role`. Recreated RLS policies: users can SELECT/UPDATE their own row (`auth.uid() = id`); service_role has unrestricted access. Pro-status 503s fully resolved.
  - **Pro-status error logging**: `logger.error` now logs full exception message (not just type name) for easier future debugging.

- **Guided 5-step workspace redesign (session 12)**:
  - `DashboardTab.tsx` — deprecated (no longer imported); replaced by `SetupStep` inline in WorkspacePage
  - `WorkspacePage.tsx` fully rewritten: 5-step indicator bar (dots + lines, green when done) + minimal nav row (logo, PRO badge, email, theme, sign out) + `SetupStep` for step 1 (resume upload zone + JD textarea + optional company/targetRole) + contextual `NextBanner` at bottom (always tells user exactly what to do next)
  - NextBanner logic: no resume → "Upload" | step 1 no JD → disabled | step 1 + JD → "Run ATS Scan" | step 2 → "Optimize →" | step 3 → "Generate Cover Letter →" | step 4 → "Optimize LinkedIn →" | step 5 done → "All done 🎉"
  - JD drawer removed — JD textarea lives inline on Step 1
  - Mobile: step number bar at bottom replaces old icon tab bar
  - 0 TypeScript errors

- **2026 audit — all 9 service fixes deployed** (session 18):
  - `pdf_service.py`: EMERALD NameError fixed
  - `models/tools_models.py`: Pydantic Field constraints added
  - `semantic_ats_service.py`: Anthropic tool use for guaranteed JSON + prompt caching (`cache_control: ephemeral`)
  - `resume_parser.py`: phone extraction, fuzzy heading matching, location parsing
  - `supabase_service.py`: httpx connection pool singleton (eliminates per-request TLS handshake)
  - `resume_service.py`: system/user prompt split on all 6 Claude call sites + prompt caching
  - `ats_service.py`: NLTK SnowballStemmer (graceful fallback), 13-industry expansion, score fix for two-generic match
  - `match_intelligence.py`: word-boundary regex (`\b`) replacing bare `in` check
  - `exceptions.py`: typed subclasses (`AIRateLimitError`, `AIAuthError`, `AIConnectionError`, `AITimeoutError`)
  - `requirements.txt`: `nltk>=3.9.0` added
- **Keep-warm endpoint**: `GET /ping` → `{"ok": true}` — ultra-lightweight, no DB calls
- **UptimeRobot**: HTTP monitor pinging `/ping` every 5 minutes — prevents Render free-tier cold starts
- **Correct Render backend URL**: `https://ai-resume-studio-api.onrender.com` (not `ai-resume-platform`)
- **Google sign-in**: added to LoginPage + SignupPage (UI wired; requires Google Cloud Console + Supabase provider setup to activate)

### 📋 Production-readiness audit (session 19)
Full audit saved at `AUDIT-2026-Production-Readiness.md` (8 categories + P0/P1/P2 plan). Key correction: the codebase is further along than CLAUDE.md implied — **API v1 versioning, DOCX download, and Interview Prep are all already implemented** (were listed "deferred").

### ✅ Done — session 19 (P0 fixes shipped in code)
All verified: `npx tsc --noEmit` clean (0 errors), backend `py_compile` clean.
- **Session persistence** — `WorkspacePage.tsx` now persists the Step-1 setup fields (`resumeText`, `resumeFileName`, `jobDescription`, `companyName`, `targetRole`) to `localStorage` under the `ars:` prefix and rehydrates on mount via lazy `useState` initializers. A refresh no longer wipes the user's setup. (Streaming *results* — scan/optimize/cover/linkedin — are still in-memory only; persisting those is P1.)
- **Pro-status loading guard** — added `proLoading = proQuery.isLoading`. While pro-status loads, gated tabs (cover-letter/linkedin/interview) render `LoadingCard` and the NextBanner shows "Checking your plan…" instead of defaulting to the "Upgrade to Pro" paywall. Paying users no longer flash the upgrade wall.
- **`STRIPE_YEARLY_PRICE_ID` added to `REQUIRED_ENV_VARS`** (`main.py`). ⚠️ **ACTION REQUIRED:** confirm `STRIPE_YEARLY_PRICE_ID` is set on Render **before the next deploy** — production startup now intentionally `sys.exit(1)`s if it's missing/placeholder. (Desired fail-loud behavior, but an unset value will refuse to boot.)
- **Fabricated social proof removed** (`LandingPage.tsx`) — deleted the three fake named testimonials ("Marcus T." etc.) and the fake "Users landing jobs at Google/Amazon/…" logo band. Replaced with honest "Early access / Be one of our first users" framing and factual capability chips. (FTC endorsement-guide risk removed.)
- **Feedback card wired** — SummaryTab "Did you hear back?" buttons now POST to new `POST /api/v1/user/feedback` (`main.py` `user_feedback` + `supabase_service.log_scan_feedback`, non-blocking). Captures `{outcome, companyName, beforeScore, afterScore}` — the outcome-labeled data the optimizer self-learning roadmap needs. Optimistic UI: thank-you shows regardless of network result.

### 🗄️ SQL migrations to run in Supabase (required for full effect)
```sql
-- 1. Feedback table (REQUIRED for the wired feedback card to persist)
CREATE TABLE IF NOT EXISTS public.scan_feedback (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  outcome text NOT NULL,               -- 'response' | 'no_response'
  company_name text,
  target_role text,
  before_score int,
  after_score int,
  created_at timestamptz NOT NULL DEFAULT now()
);
ALTER TABLE public.scan_feedback ENABLE ROW LEVEL SECURITY;
GRANT SELECT, INSERT ON public.scan_feedback TO service_role;

-- 2. Index for the upcoming scan-history read (P1)
CREATE INDEX IF NOT EXISTS idx_scan_results_user_id ON public.scan_results(user_id);
```
Until migration #1 runs, the feedback POST returns 200 but the insert is logged-and-dropped (non-blocking by design). No user-facing breakage.

### ✅ Done — session 19 cont. (analytics + monitoring shipped in code, env-gated)
- **PostHog product analytics** — `frontend-app/src/services/analytics.ts` (privacy-conscious wrapper: autocapture off, session recording off, `localStorage` persistence, identified-only profiles). Init in `main.tsx`; identify/reset wired in `AuthContext`. Funnel instrumented: `signup_completed`, `resume_uploaded`, `scan_run`, `optimize_run`, `cover_letter_generated`, `linkedin_optimized`, `interview_prep_run`, `upgrade_clicked`, `upgrade_completed`, `download_pdf`, `download_docx`. **No-op until `VITE_POSTHOG_KEY` is set on Vercel.** Privacy Policy §8 updated to disclose PostHog accurately.
- **Sentry error monitoring** — frontend `@sentry/react` init in `main.tsx` behind `VITE_SENTRY_DSN`, plus `ErrorBoundary.componentDidCatch` reports caught render errors; backend `sentry-sdk[fastapi]` in `main.py` behind `SENTRY_DSN` (added to `requirements.txt`). Both **no-op until the DSN is set**; `send_default_pii=False` on both.
- Verified: `npm install` clean, `npx tsc --noEmit` 0 errors, backend `py_compile` clean.
- **Stripe/Supabase dashboard actions completed by Akil**: yearly $144/yr price created + `STRIPE_YEARLY_PRICE_ID` set on Render; one-time price corrected to $99 (old $49 archived) + `STRIPE_ONETIME_PRICE_ID` repointed; `scan_feedback` table + `scan_results` index SQL run.

### 🔑 Analytics + monitoring env vars — SET & LIVE (session 19)
- **Vercel**: `VITE_POSTHOG_KEY` (PostHog US cloud — so no `VITE_POSTHOG_HOST` needed) and `VITE_SENTRY_DSN` set; frontend redeployed to bake them in.
- **Render**: `SENTRY_DSN` set on `ai-resume-studio-api` (auto-redeployed).
- Sentry projects: `ai-resume-frontend` (React) + `ai-resume-backend` (FastAPI), org `forge-5u`.
- PostHog: US region, project id 495758.
- ✅ Verify events are flowing: run a scan on the live site, check PostHog → Activity → Live for `scan_run`.

### ✅ All P0 complete
- **Google sign-in — LIVE.** Google Cloud Console OAuth client created (Web application; Supabase callback URL registered as the authorized redirect URI) and Client ID/Secret saved in Supabase → Authentication → Sign In / Providers → Google. Tested end-to-end on the live site — "Continue with Google" round-trips through Supabase and signs the user in. No code change was needed (`signInWithGoogle` was already wired).
- Analytics + Sentry are shipped in code and dormant until their env vars are set (see "Env vars to set when ready" above).

### ✅ P1 — SHIPPED & DEPLOYED (session 20). Verified: `npx tsc --noEmit` 0 errors, backend `py_compile` clean; committed + pushed to `main` (auto-deployed Render + Vercel).
1. **Scan history** — `GET /api/v1/resume/history` (`get_scan_history`, RLS path: anon key + user JWT, same as tracker) + "Recent scans" (last 3) on Step 1 via `historyQuery`. ⚠️ needs RLS SELECT policy on `scan_results` (SQL below).
2. **Streaming results persisted** — `scanResult`, `optimizeResult`, `optimizedScore`, `coverLetter`, `linkedin`, `interviewResult` now saved to `localStorage` (`ars:` prefix, JSON) and rehydrated on mount. A refresh keeps full output, not just setup.
3. **"Add to tracker"** — button in `SummaryTab` POSTs to `/api/v1/tracker` (company/role/status=applied/date), toasts, navigates to `/tracker`. Emits `added_to_tracker` PostHog event.
4. **Lifecycle emails (Resend)** — `services/email_service.py` (env-gated on `RESEND_API_KEY`; no-op until set). Welcome email wired: `POST /api/v1/user/welcome-email` (idempotent via `profiles.welcome_sent`), fired once from `WorkspacePage` (guarded by `ars:welcomed`). Reengagement + Pro-nudge templates written but **not yet triggered** — they need a scheduled job (Render cron) + last-activity tracking; deferred sub-item.
5. **Billing portal** — `POST /api/v1/payments/create-portal-session` (`stripe.billing_portal.Session`) + "Billing" button in workspace header (Pro + desktop only). ⚠️ needs Stripe Customer Portal activated in dashboard (Settings → Billing → Customer portal).
6. **Mobile nav labels** — `Optimize`/`LinkedIn`/`Interview` (dropped `Opt.`/`LI`/`Prep`); header logo now uses `var(--navy)`/`var(--success)` via inline `style.fill` (theme-aware).
7. **Retries** — `tenacity` retry (3 attempts, exp backoff) on transient Supabase read failures (`_http_get_retry`, used by pro-status/history/customer-id); explicit `max_retries=3` on the Anthropic client. `tenacity>=8.2.0` added to `requirements.txt`.
8. **GDPR deletion** — `DELETE /api/v1/user/account` (`delete_user_account` clears scan_results/scan_feedback/job_applications/profiles via service key; `delete_auth_user` via Supabase admin API) + "Delete account" link in workspace footer (confirm → delete → clears `ars:` localStorage → signOut → home). ⚠️ verify admin-API auth-user deletion works with the `sb_secret_*` key format in prod.
9. **Terms checkbox** — required agree-to-Terms/Privacy checkbox on signup; submit disabled until checked (replaced the passive agreement text).

### 🗄️ SQL migrations to run for P1 (Supabase SQL Editor)
```sql
-- Scan history read (item 1): let users read their own scan_results via RLS
ALTER TABLE public.scan_results ENABLE ROW LEVEL SECURITY;
DROP POLICY IF EXISTS "Users read own scans" ON public.scan_results;
CREATE POLICY "Users read own scans" ON public.scan_results
  FOR SELECT USING (auth.uid() = user_id);
GRANT SELECT ON public.scan_results TO authenticated;
-- (writes stay on the service key, which bypasses RLS — no insert policy needed)

-- Welcome-email idempotency flag (item 4)
ALTER TABLE public.profiles ADD COLUMN IF NOT EXISTS welcome_sent boolean NOT NULL DEFAULT false;
```

### ⏸️ ON HOLD — activate later (deferred by Akil, session 20)
These are dashboard/env activations for already-shipped code. The code is dormant and harmless until switched on — no breakage in the meantime. Revisit when ready.

- **Resend emails (item 4)** — ON HOLD. Blocked on a sending domain. When ready: verify a domain in Resend, create an API key, set `RESEND_API_KEY` + `EMAIL_FROM` on Render. Until then the welcome email is a silent no-op. (Don't bother with the `resend.dev` sandbox sender — it only delivers to your own address.)
- **Stripe Customer Portal (item 5)** — ON HOLD. Activate in Stripe Dashboard → Settings → Billing → Customer portal (live mode), enable Cancel + Update payment method, Save. Until then the "Billing" button returns a Stripe error for Pro users.
- **Timed lifecycle emails (item 4)** — ON HOLD (also depends on Resend above). Needs a scheduler (Render cron hitting a new endpoint) + a `last_active`/activity signal before the 48h re-engage / 7-day Pro-nudge sends can fire. Templates already written in `email_service.py`.

### ✅ P1 SQL — RUN (Akil, session 20)
- RLS SELECT policy + `GRANT SELECT` on `scan_results` — DONE. Scan history read is live (users read their own rows via `auth.uid() = user_id`).
- `profiles.welcome_sent boolean` column — DONE. Welcome-email idempotency guard active.

### ✅ Legal rewrite — Terms + Privacy (session 20). Verified `npx tsc --noEmit` 0 errors.
Rewrote both legal pages for a US Delaware C-corp operating globally (were T&T-only and stale). Draft only — **requires attorney review**; both files open with an "ATTORNEY REVIEW CHECKLIST" comment block.
- **PrivacyPage.tsx** — multi-jurisdiction (GDPR/UK + CCPA/CPRA primary; T&T + Canada PIPEDA regional). Full data inventory, GDPR legal bases, Art. 22 "you're in control / not an automated decision" AI framing, special-category resume-data handling, all 10 subprocessors, SCC-based international transfers, retention + criteria, full rights (incl. "we don't sell/share" + GPC), consent-based analytics, marketing vs transactional email, 72-hour breach notice, children under 18, EU/UK rep placeholder.
- **TermsPage.tsx** — Delaware governing law + informal-resolution → binding individual arbitration + class-action waiver + 30-day opt-out + consumer carve-out. Corrected pricing ($19/mo, $144/yr, **$99** one-time — was showing stale $49; added yearly), "lifetime" defined, auto-renewal disclosure, EU/UK 14-day withdrawal + immediate-performance waiver, strengthened AI/no-guarantee disclaimer, IP license (user owns inputs/outputs), indemnification, DMCA.
- **Cookie consent (NEW code):** `ConsentBanner.tsx` + gating in `analytics.ts` (`isEeaUkVisitor` via timezone, `needsAnalyticsConsent`, `setAnalyticsConsent`). PostHog now **does not load for EEA/UK visitors until they Accept**; non-EEA unaffected. Wired into `AppShell`.
- ⚠️ **Before publishing:** fill every `[PLACEHOLDER]` (entity name, Delaware address, interim operator, EU/UK reps, arbitration administrator, DMCA agent), confirm `privacy@`/`legal@airesumestudio.com` mailboxes exist, and get an attorney to confirm the 5–6 flagged high-risk clauses.

### ✅ UX polish — single CTA per tab (session 20). tsc 0 errors.
Tabs 3–6 (Optimize/Cover/LinkedIn/Interview) had a redundant in-tab "generate" button duplicating the NextBanner CTA. Removed the empty-state primary buttons so the banner is the single primary CTA on every tab (matches tabs 1–2 / ScanTab, which points to the banner). Kept Regenerate, download PDF/DOCX, copy, view toggles, and the free-tier upsell (not redundant). Removed the now-dead `hasResume` prop from those 4 tabs (+ `onRun` from OptimizeTab, its only user) and their WorkspacePage call sites. Optional inputs (cover-letter company, LinkedIn target role) stay in the empty state and feed the banner action.

### ✅ Header redesign + name capture (session 20). tsc 0 errors. Research-backed (separate nav from utilities; account menu; avatar initials).
- **Account menu** — new `features/workspace/AccountMenu.tsx` (avatar w/ initials + dropdown). Consolidated the header's loose cluster (PRO badge, email, theme, Billing, Sign out) into one menu: identity row (name + PRO, or **"Upgrade to Pro"** for free users), Billing/plan (Pro only), Appearance (theme), Delete account (moved out of the footer), Sign out. Header went from 7 items → logo · Applications · avatar.
- **Tracker → "Applications"** — renamed, now a single labeled nav item (briefcase icon) in the header.
- **Tools folded into Optimize** — the old header "Tools" button + standalone `tools` tab render are gone; `ToolsTab` now renders inside `OptimizeTab` as a collapsible "Writing tools" `<details>` (bullet enhancer + summary generator), contextual to editing. Removed `ToolsTab` + `IconSun/IconMoon` imports from WorkspacePage.
- **Name capture at signup** — `SignupPage` now collects a required **First name** + an optional **marketing opt-in** checkbox (GDPR-clean basis for the re-engage/Pro-nudge emails). `AuthContext.signUp(email, password, meta)` passes `{ full_name, marketing_opt_in }` into Supabase `user_metadata` (no new DB column). Header/avatar read `user.user_metadata.full_name` (Google users get it automatically), falling back to the email prefix + first-letter initial.
- **Deferred (progressive profiling):** job-search-status + "how did you hear about us" — collect in a later optional first-run step, not the signup form (research: minimal fields = higher conversion).

### ✅ Unified header + Optimize-tab flow (session 20). Verified `npx tsc -b` 0 errors + full `vite build` green. (NOTE: verify with `tsc -b`, NOT `tsc --noEmit` — the latter misses project-mode errors and caused a red Vercel deploy earlier.)
- **Shared `components/SiteHeader.tsx`** — one static header for the whole app, auth-aware. Signed-out: logo · Pricing · theme · Sign in · Get started. Signed-in: logo(→/workspace) · **Workspace** · **Tracker** · account menu. Self-contained (own pro-status query, billing + delete handlers). Applied to LandingPage, PricingPage, TrackerPage, WorkspacePage — removed each page's bespoke header + now-dead imports (useTheme/IconSun/IconMoon/AccountMenu/Logo/signOut where unused).
  - Fixes: landing no longer shows "Sign In" when authenticated + now has a way back to /workspace; Tracker header matches the app; consistent look across pages. Rename resolved: two clear items — **Workspace** (editor) + **Tracker** (was the mislabeled "Applications", which pointed at /tracker).
  - ⚠️ Still bespoke (not yet on SiteHeader): PrivacyPage, TermsPage (logo + Back nav), LoginPage, SignupPage (centered logo-only). Fine as-is; unify later if wanted.
- **Optimize tab reordered** for end-to-end flow (NN/g progressive disclosure): **Results → Your Optimized Resume → Refine (collapsed: Skills-First + Writing tools) → Download**. Previously Download sat *before* the refine controls (Skills-First + Writing tools), so users hit refinement after the finish line. Skills-First was extracted from the Download card into a grouped, collapsed "Refine your resume" `<details>` that now precedes Download. (Optimization-guidance box still trails, conditional/advisory.)
- User chose a **static** (non-sticky) header.

### ✅ "New scan" reset (session 20). tsc -b + vite build green. Research-backed (NN/g: preserve costly input, friction on destructive actions).
Persistence meant a returning user's old resume/JD/results reloaded with no obvious way to start fresh. Added:
- **`SiteHeader` optional `onNewScan` prop** → renders a "New scan" button in the workspace header (only when passed; WorkspacePage passes `handleNewRole`). Not shown on other pages.
- **`handleNewRole` upgraded**: now also clears `companyName` + `targetRole` (not just JD/results), **keeps the resume** (costly to re-enter — the `Change` link on Step 1 remains the deliberate resume-swap), and shows a **lightweight `window.confirm`** only when there's generated work to lose.
- **Auto-clear stale downstream results on re-scan**: `scanMutation.onSuccess` now nulls optimize/optimizedScore/cover/linkedin/interview so a fresh scan never sits next to an old optimized resume.

### ✅ Optimize tab — editable resume (Option B, session 20). tsc -b + vite build green. Research-backed (human-in-the-loop editing; orphaned-tool anti-pattern).
Optimized resume was read-only and the page carried orphaned Writing Tools (bullet enhancer + summary generator produced copy-only text with nowhere to go, partly duplicating the optimizer). Implemented Option B:
- **Editable optimized resume** — the "Optimized" view is now an editable `<textarea>` bound to new `editedResumeText` state. **Copy + PDF/DOCX downloads use `editedResumeText`** (the working copy), not the raw AI text. "Reset to AI version" link + helper copy + an honest note when edited ("score reflects the AI version; re-run the scan to update"). `editedResumeText` lives in `WorkspacePage` (persisted with the `ars:` results, seeded on optimize, cleared on new scan/upload/New-scan).
- **Removed Writing Tools from the Optimize page.** `ToolsTab.tsx` + the `/tools/*` endpoints remain in the codebase (unused, reusable for a future standalone tools entry point) — just not rendered.
- **Skills-First simplified** to "Generate / Use skills-first layout" which **loads into the editor** (single source of truth); dropped `activeVersion`/`displayedResumeText` and the version-toggle + the collapsed "Refine" wrapper.
- **Deferred (v1.5):** a "Re-score my edits" button (re-run the semantic scan on edited text to refresh the after-score). Shipped the honest note instead — measure whether users edit before building it. Rich/section-aware editor = Option C, later.

### ✅ Recent scans → restorable (Option 3, client-side) + paste-to-replace resume (session 20). tsc -b + vite build green. Research-backed (NN/g recognition-over-recall + flexibility; ICO/GDPR data-minimization).
- **Recent scans is now clickable to reload a past scan** — but **client-side only** (privacy-first). Key reason: `scan_results` (server) only stores scores/keywords/role/date, **never the resume or JD**, so server history was never restorable. Rather than persist full resumes server-side (ICO data-minimization risk — resumes can hold special-category data), we keep the last **5 full sessions in the browser** (`ars:sessions`, `ars:currentSessionId`). A `SavedSession` snapshot (resume + JD + company/role + scan/optimize/edited/cover/linkedin/interview) is upserted continuously while a scan exists; "New scan" starts a new session id (archiving the current); clicking a Recent scan restores everything and lands on the Scan tab. Emits `recent_scan_restored`.
  - Trade-off (documented): history is **per-device**, not synced. Cross-device restore would require Option 2 (store inputs server-side + retention window + Privacy Policy update) — deferred.
  - `WorkspacePage` no longer calls `getScanHistory` for this UI (the `GET /api/v1/resume/history` endpoint + `scan_results` logging remain for analytics/future, just unused by the Recent-scans list).
- **Paste-to-replace resume** — the resume card's single "Change" (file-only) is now **"Upload new" + "Paste new"** (`showPaste` toggle reveals the paste box). Fixes the gap where, after "New scan," a user couldn't paste a *different* resume for a new JD (NN/g flexibility & efficiency).

### ✅ Optimizer honesty — Fix A + B (session 20). tsc -b + py_compile clean. Research-backed (NN/g visibility-of-system-status; SHRM resume-fabrication consequences).
Aerospace test (Marcus Delgado) exposed two issues: the "What changed" chips claimed keywords not actually in the resume (Six Sigma, Teamcenter), and the optimizer added an unsupported **Nadcap** skill + an invented **DFMEA** activity bullet.
- **Fix A (chips honesty, `OptimizeTab.tsx`)** — `addedKeywords` now = a previously-missing keyword that **literally appears in the optimized text and didn't before** (case-insensitive; matches phrase or parenthetical acronym), instead of the re-scorer's `missingBefore − missingAfter` opinion. Chips can no longer claim something not in the document.
- **Fix B (optimizer guardrail, `OPTIMIZER_SYSTEM_PROMPT` in `resume_service.py`)** — the "never infer" net only covered *named tools/software*; certs/standards/methodologies fell into category A. Now: (1) never-infer rule explicitly covers **certifications, licenses, standards, accreditations, regulatory/quality frameworks (AS9100, Nadcap, ISO, ITAR…), and named methodologies (Six Sigma/Black Belt, DFMEA, PFMEA, 8D, Kaizen, Scrum…)** — skip unless the exact term is in the original; (2) new **NEVER INVENT ACTIVITIES** rule (no manufactured bullets); (3) Skills section may not inject unsubstantiated keywords; (4) A/B split + examples updated to aerospace.
- **Expected effect:** honest but *smaller* score gains — correct trade vs. fabrication that gets offers rescinded. Category A (reframe real work) still allowed.
- **B+ deferred** (user-facing "verify these" flag): promote `check_fabricated_credentials` (currently log-only, regex `_COMPILED_CREDENTIALS`) to a UI callout above the editable resume — only after tuning vs. real output so it doesn't false-positive on legit reframes.
- **Validate after deploy:** re-run Marcus (already loaded) → Nadcap/DFMEA/Six Sigma/AS9100/8D gone unless in his original, chips honest, smaller +score. Then test other industries (healthcare certs/EHR, software AWS/Scrum, finance CPA).

### ✅ Fix B v2 — Piece 1 (v2.1 prompt) + Piece 2 (suggestion UI) (session 20/21). Verified: `tsc -b` 0 errors + full `vite build` green; backend `py_compile` clean + function unit-tested. Research-backed (2 sources each: Claude/AWS prompt best-practices, faithfulness/grounding, SHRM/CareerClimb resume ethics, TalentGuard/Centranum proficiency ladder, HHS/FASB obligation-vs-credential, aiuxdesign/AufaitUX HITL).
Three-persona test (Priya nurse / Daniel software / Sofia accountant) after Piece 1 showed the **dangerous** fabrications all died (hemodynamic monitoring, titrated infusions, "Telemetry" title, "internal controls documentation", "no material findings", CPA/NetSuite/ERP) but **reworded** climbs survived ("control environment" = internal controls in disguise, "no carryover discrepancies" = invented outcome reworded, agile, CI/CD). Root cause: example-based rules overfit → model dodges via synonyms.
- **v2.1 (`resume_service.py`, `OPTIMIZER_SYSTEM_PROMPT`, prompt-only):** (1) **Proficiency-ladder rule** (awareness→novice→proficient→advanced→expert) as the domain-independent enforcement — "name a competency only at the rung the original evidences," works for every industry not just the 3 tested. (2) **Concept-not-string rule** — "judged on the concept, not the exact words; rewording a blocked concept doesn't unblock it," with control-environment / invented-outcome-rewording called out. (3) Example table **relabeled "illustrative, not exhaustive"** (+ agile/Scrum + "no carryover discrepancies" rows) to stop narrow pattern-matching. (4) **Verification pass** swapped to the concrete **third-party test**: "Could a former employer or licensing board confirm this from the candidate's actual record?"
- **Piece 2 (5 files) — gray-zone standards → opt-in suggestions, never auto-inserted:**
  - `resume_service.py` — new **pure-code** `suggest_universal_standards(resume, jd, optimized)` (no AI, no network, unit-tested). Conservative **obligation-only allowlist**: `healthcare`→HIPAA, `accountant`/`finance_manager`→GAAP. Returns `[{term, why}]` only when the JD names it AND it's absent from both resume and optimized text. **Never** suggests acquired credentials (CPA/BLS/licenses/ERP) — grounded in HHS (HIPAA binds all covered-entity workforce) vs FASB/SEC (GAAP is the reporting framework, CPA is a separate license).
  - `main.py` — `suggestedStandards` added to the SSE `result` payload (no new endpoint, zero extra API cost).
  - `resumeApi.ts` — `suggestedStandards: {term, why}[]` on `OptimizeResult` + SSE parser.
  - `OptimizeTab.tsx` — opt-in **"Standards you likely qualify for"** card (amber, distinct from green "added" chips): each row = `+ TERM` button + one-line "why," default OFF, click inserts into the Skills section of `editedResumeText` and flips to "✓ Added." HITL: reject = don't click. Non-destructive `insertStandardIntoResume` helper.
  - `analytics.ts` — `standard_suggested` (on result) + `standard_accepted` (on click) events (permissive `track(event, props)`, no union to extend).
- **Validate after deploy:** re-run Priya/Daniel/Sofia → "control environment", "no carryover discrepancies", agile, CI/CD gone; category-A reframes survive; HIPAA (Priya) + GAAP (Sofia) now appear as opt-in suggestion chips instead of silent insertions. Then test a 4th untested industry (e.g. legal, trades) to confirm the ladder generalizes.
- **Expandable:** `_UNIVERSAL_STANDARDS` allowlist is intentionally tiny (HIPAA, GAAP) — add more obligation-only standards as new roles are validated.

### ✅ Fix B v2.2 — GAAP-sync + summary discipline (session 21). `py_compile` clean + function unit-tested. Research-backed (Indeed/Jobscan: summary distills proven experience; LinkedIn keyword guide disconfirming search: each summary keyword must map to a bullet).
Six-persona test (3 tuned + 3 new industries) showed v2.1 blocks named tools/certs/licenses/title-promotion **100% across every industry** (marketing SaaS tools, electrician Master license/OSHA/NFPA, HR SHRM/Workday all excluded; Victor even declined to inflate + let score drop rather than fabricate). Two residual patterns remained: (1) **GAAP still written into text** — root cause: prompt never-infer list named HIPAA but not GAAP, so GAAP-in-text *suppressed* its own Piece 2 chip (Piece 2 only fires when the term is absent). (2) **Climbs now cluster in the SUMMARY** (Sofia "control environment", Amara "performance management", Rebecca "lead generation" / "marketing analytics") — free prose evades the item-by-item verification pass.
- **GAAP-sync (`resume_service.py`):** added `GAAP, GAAS` to both never-infer standards lists so the optimizer keeps them OUT of the text → Piece 2 surfaces GAAP as an opt-in chip (like HIPAA). Added a **SYNC RULE comment** on `_UNIVERSAL_STANDARDS`: every allowlist `term` must also be blocked in the prompt or its suggestion is suppressed.
- **Summary discipline (`OPTIMIZER_SYSTEM_PROMPT`, SUMMARY ALIGNMENT):** new rule — every competency/skill/standard named in the summary MUST appear in and be supported by ≥1 experience bullet, at the same proficiency rung; apply the third-party test to the summary word by word; banned-climb examples (performance-review coordination ↛ performance management; reconciled accounts ↛ control environment; ran campaigns ↛ lead generation).
- **Validate after deploy:** re-run Sofia → GAAP now a chip not text, "control environment" gone from summary; re-run Amara → summary says "performance-review coordination" not "performance management"; re-run Rebecca → "lead generation" gone.

### 📄 Engine review docs (session 21) — research-backed reviews of the two MVP engines, written to repo (specs, mostly not yet built)
`ENGINE_REVIEW.md` (ATS scorer + optimizer + the "real gatekeepers are knockouts + parsing, not a keyword score" reframe), `FABRICATION_CHECKER_SPEC.md` (MiniCheck/HHEM two-layer checker — gated on user evidence + plan upgrade), `PARSE_FRIENDLINESS_SPEC.md`, `GENERATOR_FAITHFULNESS_REVIEW.md`, `SCORE_CALIBRATION_SPEC.md`, `MODEL_PINNING_SPEC.md`, `SCORER_PRECEDENCE_SPEC.md`, `RESEARCH_PROTOCOL.md`. Most are a de-risked backlog; the highest-value next action remains getting the 3 beta users' feedback, not more building.

### ✅ Eval harness scaffold — Build A shipped (session 21). Test-only; NO engine code touched. Verified: `py_compile` clean (all 5 modules) + all 6 deterministic checks PASS in-sandbox (no API key).
The regression gate that ends the anti-fabrication whack-a-mole. Lives in `backend/tests/eval/` (see its `README.md`). Spec = `PRODUCTION_ENGINE_SPEC.md` §2.
- `personas.json` — golden set, **6 high-signal personas** covering every failure class found in live testing: `priya_nurse` (clinical-competency climb + title), `daniel_software` (named cloud/tools + agile/CI→CI/CD), `sofia_accountant` (certs + SOX/internal-controls rewording + GAAP-in-text), `marcus_supply_chain` (named tools/certs + Excel→pivot-tables feature-elaboration + skill fabrication), `ricardo_sales` (title promotion to target role + named CRM/methodology), `devon_data_analyst` (qualifier-stripping SQL basic→SQL + named tools). Each: `{resume, jd, must_not_contain[], title_must_stay, must_preserve?}`. **Target-role titles are listed in `must_not_contain`** so promotions are caught as banned terms (no fragile headline parsing). Every future bug → append a persona.
- `_eval_core.py` — `check_persona()` (pure/deterministic gate: banned-term + `detect_unsupported_skills` reuse + original-title-present + preserved-qualifier) and `run_optimizer()` (live: `calculate_ats_score` → `stream_resume_optimization`, mirrors prod).
- `test_detector.py` — deterministic checks, **no API key, always runs** (this is the CI gate). `test_optimizer_faithfulness.py` — live per-persona eval, `skipif` no `ANTHROPIC_API_KEY`. `run_baseline.py` — runs N×/persona, writes `baseline.json` leak-rate. `conftest.py` — puts `backend/` on `sys.path`.
- **Deliberately did NOT build Build B (verify→regenerate loop) yet** — the sequence requires the baseline number FIRST so B's delta is measurable; B also touches the money-path optimize endpoint + carries the unresolved streaming tradeoff (buffer-verify-then-stream vs targeted regen) that's Akil's call. Sandbox can't run the live LLM anyway.
- ⚠️ **NEXT ACTION (Akil, on Mac):** `cd backend && ANTHROPIC_API_KEY=... python -m tests.eval.run_baseline 3` → get the real "before" leak rate. Then build B against that number. (Also: `pytest tests/eval/test_detector.py -v` runs free anytime; commit the new files with the usual `rm -f .git/*.lock` prefix.)

### ✅ First baseline run + harness/prompt fixes (session 21). Akil ran `run_baseline 3` on his Mac. Verified: `py_compile` + deterministic split checks PASS in-sandbox.
Raw first number: **0.667 (12/18)** — but decomposed, most were NOT real fabrications. Findings + fixes:
- **Harness had two false-positive modes → fixed (test-only):** (1) `must_not_contain` did naive substring matching → **"Outreach" (the tool) collided with the common word "outreach"**; dropped it from Ricardo + switched banned-term matching to **word-boundary regex** (`_contains_term`, kills "cte"-in-"protected" etc). (2) The conservative head-word `detect_unsupported_skills` was gating CI on **legitimate reframes** ("broke a monolith into services" → "service decomposition", "supplier follow-up", "continuous patient monitoring"). **`check_persona` now returns `{"hard":[...], "soft":[...]}`** — HARD (named tools/certs/titles + dropped title/qualifier) is the CI gate; SOFT (head-word advisory, same as the prod "double-check these" callout) is reported, never gated. `run_baseline` now reports both rates separately; `test_optimizer_faithfulness` asserts on HARD only.
- **Real product bug the harness exposed → fixed (prompt-only, `OPTIMIZER_SYSTEM_PROMPT`):** on a big career-change gap (Devon teacher→senior-analyst), the optimizer appended an **explanatory note INTO the resume body** ("A note on this resume's ATS score… none of which appear in the original…"). A real user would see an essay pasted onto their resume, and it tripped every checker (banned terms quoted inside the model's own note). Root cause: line 254's "say so honestly" was read as license to write a note. Fix: reworded "say so honestly" → "express ONLY through restraint… NEVER add a note/disclaimer/explanation"; hardened the final OUTPUT rule ("first characters must be the candidate's name; last must be the final resume line; no meta-commentary before, after, or inside").
- **Real residual leak (mild/gray):** Sofia **GAAP written into Skills 2/3** despite GAAP being in the never-infer list. Gray because her financials almost certainly ARE GAAP — which is exactly why v2.2 made GAAP an opt-in Piece 2 *chip*, not auto-text. The suppression isn't fully holding; the verify→regenerate loop (Build B) is the deterministic backstop for this class (+ Devon's residual tool leaks after the commentary fix).
- ⚠️ **NEXT (Akil):** `pip install tenacity` in the venv first (the run showed `keyword_intelligence: ModuleNotFoundError: tenacity` → it was skipping the keyword-injection that prod uses, so the local eval wasn't prod-identical), then re-run `python -m tests.eval.run_baseline 3` for the **clean HARD-leak number**. That trustworthy number is the "before" for Build B.

### ✅ Clean baseline result (session 21): **raw hard-leak rate = 0.111 (2/18), 100% Sofia GAAP.** Devon career-changer went catastrophe→clean 3/3 after the commentary fix. Soft-flag rate 0.222 (all legit reframes: service decomposition, full-cycle sales, senior stakeholder communication — advisory, not gated). The one residual is GAAP written into Sofia's Skills — mildest class (her financials likely ARE GAAP; that's why it's designed as an opt-in chip), and prompt-suppression (v2.2) has now failed on it twice → deterministic loop is the right tool.

### ✅ Build B shipped — verify→regenerate loop (session 21). User chose **"buffer, verify, then stream"** (correctness-first). Verified: `py_compile` (services + main + eval) clean; verifier logic unit-tested in-sandbox (catches GAAP+CPA absent-from-original, stays clean when GAAP in original, SOX word-boundary guard holds). ⚠️ Live LLM path NOT runnable in sandbox — Akil must measure on Mac.
- **The loop (`resume_service.py`):** `generate_verified_optimization(resume, jd, missing, score, max_regens=2)` → buffers the draft (via `stream_resume_optimization`, no live stream), runs the **external deterministic verifier**, regenerates with the exact flagged terms named if needed, degrades gracefully to the best draft after the cap. Returns `{text, regens_used, residual_fabrications, first_draft_flags}`. Research rule honored: loop is driven by the EXTERNAL verifier, never model self-critique.
- **The verifier = `verify_optimization` = `check_fabricated_credentials` (certs/degrees) + NEW `check_unsupported_standards`** (`_COMPILED_STANDARDS`: AS9100/Nadcap/ISO/ITAR/SOC2/HIPAA/GAAP/GAAS/IFRS/SOX/PCI-DSS/GDPR/cGMP/NFPA70E/DFMEA/PFMEA/APQP/PPAP/8D/Kaizen, word-boundary). High-precision on purpose — deliberately EXCLUDES the noisy head-word `detect_unsupported_skills` (would loop on legit reframes). `check_fabricated_credentials` alone missed GAAP — that's why the standards checker was needed.
- **Endpoint (`main.py` optimize SSE):** Step 3 now calls `generate_verified_optimization` (buffered) → emits "Double-checking for accuracy…" status if a regen happened → **replay-streams** the verified text in 48-char token chunks (frontend renders progressively, same UX shape, just after verification). Cost: buffering adds latency even on clean runs (inherent to option A, user accepted); extra Anthropic call only when flagged (~11%). Residual fabrications (survived 2 regens) folded into the existing red "double-check these" callout (`unverifiedSkills`) — no frontend change needed. Payload also gains `residualFabrications` + `verifyRegens` for observability. `correction_feedback` param threaded through `stream_resume_optimization` + `_build_optimizer_user_message` (prepended, so the model reads the constraint first).
- **Eval "after" path:** `run_optimizer_verified` in `_eval_core.py`; `run_baseline.py` now takes a mode arg — `python -m tests.eval.run_baseline 3 verified` runs the loop and writes `baseline_verified.json`, to compare against the raw 0.111.
- ⚠️ **NEXT (Akil, Mac):** (1) `python -m tests.eval.run_baseline 3 verified` → expect Sofia GAAP → ~0 (the "after"). (2) Manually optimize once on the live/local app to confirm the replay-stream UX feels right (buffer pause → text types out). (3) Commit (`rm -f .git/*.lock` prefix). Frontend change is OPTIONAL (residual already surfaces via existing callout) — a dedicated "we removed X" note is a future nicety.

### ✅ Build B measured + persona set widened to 12 industries (session 21).
- **Verified-loop "after" (Akil ran `run_baseline 3 verified`):** hard-leak **0.111 → 0.056 (1/18)**. **Sofia GAAP → clean 3/3** — the loop nailed its exact target. The one residual: Daniel "CI/CD" 1/3 — a *qualifier/practice climb* (his resume has "continuous integration," not CD), which the loop's verifier intentionally does NOT cover (it stays narrow on named creds/standards to avoid false-positives). Decision: keep the deterministic verifier high-precision; CI/CD belongs to the prompt class (low-stakes, spectrum term), not the loop (high-stakes, binary/checkable). Soft-flag rate is noisy run-to-run variance on legit reframes — advisory, not gated.
- **Persona set 6 → 12** (`personas.json`): added `elena_paralegal` (legal: e-discovery tools + notary + title promo), `victor_electrician` (trades: master license + NFPA 70E/OSHA/NEC + PLC — note NFPA 70E is in `_COMPILED_STANDARDS` so it tests the LOOP cross-industry), `tanya_teacher` (education: named curricula Fountas & Pinnell/Responsive Classroom + PowerSchool + ESOL), `rebecca_marketing` (martech HubSpot/Google Ads/GA4 + demand-gen + title promo), `grace_medical_biller` (healthcare admin: CPC/AAPC/CCS certs + Epic), `omar_finance_analyst` (CFA + Bloomberg/SQL/Tableau + title promo). Validated: JSON parses, no banned term appears in its own resume (no built-in false positives), titles present. Purpose = breadth (confirm the rules generalize), since the core classes are already suppressed.
- ⚠️ **NEXT (Akil):** re-run both baselines on the wider set — `python -m tests.eval.run_baseline 3` (raw) then `... 3 verified` — to get the 12-industry hard-leak numbers. Cost scales with personas × runs (×~1.1 for verified regens); 12×3 is still cheap.

### ✅ 12-industry baseline measured + umbrella-climb prompt rule (session 21).
- **Results (Akil ran both):** raw hard-leak **0.083 (3/36)** = Sofia GAAP×2 + Devon "business intelligence"×1. Verified **0.028 (1/36)** = Daniel "CI/CD"×1 only (Sofia GAAP → clean 3/3 again). **All 6 NEW industries (legal/trades/education/marketing/healthcare-admin/finance) = ZERO hard leaks in both runs** → the anti-fabrication rules generalize, not overfit to the original 6. `victor_electrician` clean incl. NFPA 70E (prompt suppressed it before the loop was needed). Soft-flag rate ~0.5 = noisy legit-reframe advisory, not gated.
- **The one recurring residual = the "umbrella/spectrum climb" class** (Devon: CI/CD then business intelligence; Daniel: CI/CD). The loop deliberately doesn't cover it (not a named cred/standard → high-precision verifier stays out). CI/CD + Agile/Scrum were already ✗ examples in the prompt yet still slipped, so example-matching wasn't enough.
- **Fix (prompt-only, `OPTIMIZER_SYSTEM_PROMPT`):** added a general **"NO UMBRELLA / CATEGORY CLIMB"** rule after PRESERVE SELF-LIMITING QUALIFIERS — names the *pattern* (don't roll narrow entry-level activities up into the parent discipline/named practice), with cross-industry examples (basic SQL+reporting ≠ business intelligence; continuous integration ≠ CI/CD; scheduling posts ≠ demand generation; monitoring vitals ≠ critical care). `py_compile` clean.
- ⚠️ **NEXT (Akil):** re-run `run_baseline 3` + `... 3 verified` → check Devon BI + Daniel CI/CD drop (target: verified → ~0/36). Then commit all of session 21's eval + engine work (`rm -f .git/*.lock` prefix).

### ✅ Umbrella nudge measured — did NOT move the needle; declared the floor (session 21). Honest result.
Re-ran both after the umbrella-climb rule: raw **0.083 → 0.111** (went UP: Sofia GAAP×3 + Daniel CI/CD×1), verified **0.028 → 0.028** (flat: Devon BI×1). The leaks just reshuffled across personas/terms — at n=3/persona the run-to-run variance is LARGER than the nudge's effect, so it can't be credited with anything. Kept the rule anyway (correct guidance, zero risk), but it's not a measured win.
- **What IS stable across every run:** (1) the loop reliably kills the high-stakes standards class — Sofia GAAP caught 3/3 again (raw 3/3 → verified clean 3/3); (2) the verified floor is ~0.028 (1/36) and is ALWAYS the mild umbrella/spectrum class (BI or CI/CD), whichever persona surfaces it — stochastic, gray (Devon's work arguably IS entry-level BI; Daniel DOES CI), backstopped by the human "double-check these" callout.
- **Decision: STOP prompt-tuning this class; ship.** Deterministically killing BI/CI-CD would require adding them to the loop verifier = the scope-creep + false-positive risk (stripping it from someone who genuinely does CI/CD) already rejected. Not worth it for the mildest class. To actually MEASURE any future tweak, bump to 5–10 runs/persona (n=3 measures noise) — for rigor, not safety.
- **Engine faithfulness work for session 21 is DONE.** Harness (12 personas) + verify→regen loop + prompt rules. Verified rate 0.028, all high-stakes classes ~0. ⚠️ Commit everything (`rm -f .git/*.lock` prefix): `backend/tests/eval/`, `resume_service.py`, `main.py`, `CLAUDE.md`.

### ✅ Persona set widened 12 → 21 (session 21). Validated: JSON parses, no banned term in its own resume, titles present, guard preserve-terms present.
Added 8 sector-fillers chosen to bring NEW fabrication classes (not just new domains) + 1 false-positive guard:
- `jamal_hospitality` (ServSafe cert + Toast POS + title), `lorena_quality_tech` (**AS9100 + ISO 9001 — both loop-catchable** + Six Sigma/GD&T/SPC — tests the verifier cross-industry, aerospace/manufacturing), `marcus_gov_admin` (**SECURITY CLEARANCE fabrication — new high-stakes class** + FAR + title), `priyanka_hr` (SHRM-CP/PHR + Workday + "performance management" umbrella + title), `diego_leasing` (**real-estate license** + MLS + title), `terrence_warehouse` (**CDL license** + DOT/hazmat + title), `hana_designer` (Figma/Adobe XD/Sketch + design-system/product-design umbrella + title), `aaliyah_case_manager` (**LCSW/LMSW clinical licensure** + psychotherapy + DSM-5 + title).
- **`nina_accountant_fpguard` = the false-positive GUARD** (the blind spot all prior personas shared): a genuinely CPA-licensed senior accountant whose resume really contains CPA + GAAP + NetSuite. `must_preserve: [CPA, GAAP, NetSuite]` — these MUST survive optimization; `must_not_contain: [CMA, CFA, Accounting Manager]`. Tests the OTHER failure mode — that the loop/prompt does NOT over-strip legitimate credentials (esp. GAAP, which the verifier only strips when ABSENT from the original; Nina has it, so it must stay). If GAAP/CPA get dropped → hard leak (dropped required qualifier).
- New classes now covered: professional/occupational licenses (RE, CDL, LCSW), security clearance, food-safety cert, and false-positive preservation. Loop-catchable standards now tested in 2 industries (accounting GAAP + aerospace AS9100/ISO 9001).
- ⚠️ **NEXT (Akil):** `python -m tests.eval.run_baseline 3` then `... 3 verified` on the 21-persona set → new hard-leak numbers (cost/time ~1.75× the 12-set; still minutes). Watch especially: clearance/license personas (should be clean — high stakes) and Nina (CPA/GAAP must be PRESERVED — a hard leak there means over-stripping, the opposite problem). Then commit.

### ✅ New-batch result (9 new personas measured in isolation, session 21). Added `EVAL_ONLY=id1,id2,…` env filter to `run_baseline.py` to run a subset.
Akil ran only the 9 new personas: raw **0.037 (1/27)**, verified **0.0 (0/27)**. Two key confirmations:
- **All high-stakes classes clean across the new industries (raw AND verified):** security clearance (Marcus), RE license (Diego), CDL (Terrence), LCSW (Aaliyah), Figma/design tools (Hana), and **AS9100 + ISO 9001 (Lorena) clean 3/3 even in raw** — prompt suppressed the loop-catchable standards before the loop fired. Rules generalize to gov/legal/trades/aerospace, not overfit.
- **False-positive GUARD passed:** Nina kept CPA + GAAP + NetSuite in all runs (0 dropped-qualifier leaks). Confirms the loop does NOT over-strip legit credentials — GAAP survives because the verifier only strips it when ABSENT from the original. The mechanism that kills Sofia's fabricated GAAP correctly preserves Nina's real GAAP.
- Only raw hard leak = Priyanka "performance management" (1/3) — the umbrella/spectrum class again (loop doesn't cover by design; didn't surface in verified). 0/27 verified is a strong sample, not proof of zero (umbrella class is stochastic, ~1/27 in raw). Whole 21-set is now the golden regression gate.

### ✅ Repositioning shipped — "the resume you can defend" (honesty-first) — session 21. Verified: `tsc -b` exit 0 (clean). ⚠️ `vite build` NOT runnable in sandbox (macOS rollup binaries in mounted node_modules) — confirm with `npm run build` on Mac before deploy.
Research-backed pivot OFF the "beat the ATS" myth (Enhancv 25-recruiter study: 92% of ATS don't auto-reject; the myth is often sold by resume tools) ONTO the honesty wedge (ResumeBuilder: ~41% of resume-liars get offers rescinded; career-expert 2026 consensus = "sound like you, not a language model"). Codebase already delivered this (anti-fabrication engine + 21-industry gate); the site was still selling the myth. Four moves implemented (copy/label only — no engine/logic change):
- **Move 1 — Hero (`LandingPage.tsx`):** H1 "Your Resume, Optimized for Every Job" → **"The resume you can defend."** Subhead reframed to the interview-gap promise. Hero mockup: "ATS Scan Results/STRONG MATCH" → "Recruiter Readiness/RECRUITER-READY"; score-breakdown labels → recruiter-skim language (relevance, skimmable structure, natural keywords, proof, format). Subcopy → "Won't fabricate a thing". Trust-bar chips → Won't fabricate / Sounds like you / Recruiter-skim scoring / Defensible in interviews.
- **Move 2 — New "Passes the bot. Fails the interview." section (`LandingPage.tsx`):** honest-vs-generic 2-column comparison (generic AI resume = caught in interview ✕ vs honestly optimized = defensible ✓).
- **Move 3 — Score reframe (`ScanTab.tsx`, `OptimizeTab.tsx`, `SummaryTab.tsx`):** "ATS Compatibility Report" → "Recruiter Readiness Report"; "ATS Score/ATS Cleared" → "Recruiter Readiness/Recruiter-ready"; "ATS score improvement" → "recruiter-readiness gain"; before/after labels reframed. Added an HONEST CAVEAT under the scan score: "Measures clarity and fit for the role — not a magic ATS pass. Recruiters skim fast; they rarely auto-reject." Compatibility copy de-mythed. Kept the "Run ATS Scan" action button + template "ATS-safe" parse notes (those are technically accurate, not myth). Scoring ENGINE unchanged.
- **Move 4 — Honesty as signature (`LandingPage.tsx` + `OptimizeTab.tsx`):** replaced thin "Early access" section with a **"The honesty guarantee"** 3-point proof block (won't fabricate / tested across 21 industries / you stay in control) + honest founder note (transparency-as-proof, since zero users = no testimonials — per trust research). In-product: the red "Double-check these" callout reframed to **"Honesty check — verify these before you send"** ("protecting you, not padding you"). The opt-in "Standards you likely qualify for" callout was already ideal — left as-is.
- FAQ: strengthened the no-fabrication answer + added an honest **"Will this help me 'beat the ATS'?"** answer that refuses to sell the myth. Footer tagline → "Honest AI resume optimization — sound like you, at your best."
- ⚠️ **NEXT (Akil):** `cd frontend-app && npm run build` to confirm vite build green, then commit + push (auto-deploys Vercel). Then this is the messaging to test on the 20 contacts — ask "would you pay to not get caught in an interview, and what would you use instead?" Positioning is a HYPOTHESIS until they react.

### ✅ Repositioning gap-fix — workspace + funnel consistency (session 21, from Akil's screenshots). Verified `tsc -b` exit 0.
First pass missed several surfaces still saying "ATS" + a real honesty bug. Fixed:
- **CRITICAL honesty bug:** `ScanTab.tsx` `scoreToPercentile` still returned ungrounded **"Top 5% / Bottom 25% of applicants"** percentiles (OptimizeTab was fixed in session 21, ScanTab was missed). Replaced with the same honest match-strength bands (Strong/Good/Moderate/Fair/Weak match). We have NO applicant-pool data — the percentile was a fabricated claim, directly against the positioning.
- **`ScoreRing.tsx`** default `label='ATS Score'` → `'Recruiter Readiness'` (only ScanTab uses the default; OptimizeTab passes `label=""`). Fixes the "ATS SCORE" caption under the scan ring.
- **`WorkspacePage.tsx`:** step label 'ATS Scan' → 'Scan'; NextBanner 'unlock ATS scoring'/'Run ATS Scan' → 'unlock your recruiter-readiness scan'/'Run Scan'; 'maximize your ATS match' → 'sharpens every bullet for this role — in your voice'; SetupStep subtitle 'run your ATS (Applicant Tracking System) scan' → 'run your recruiter-readiness scan'; JD placeholder 'ATS keyword scoring' → 'recruiter-readiness scoring'; ErrorBoundary tabName 'ATS Scanner' → 'Scan'.
- **Funnel surfaces:** LandingPage plan feature + FAQ + footer link, PricingPage feature labels + 2 FAQ answers → "recruiter-readiness scan". OptimizeTab locked-preview desc 'ATS score improved' → 'recruiter readiness improved'.
- **Deliberately LEFT (accurate, not myth):** template "ATS-safe format" parse notes (technically true about parsing), the landing "Will this help me 'beat the ATS'?" FAQ (intentionally de-myths), Privacy/Terms "ATS score" legal descriptions (accurate + honestly caveated), and the "Run ATS Scan"→now "Run Scan" action (kept the action, dropped the acronym). Code comments untouched.

### ✅ Quick wins shipped (session 21). Verified: `py_compile` + `tsc -b` + `vite build` green; Executive PDF smoke-tested (renders + contact extracts from body).
- **LinkedIn guardrail** (`resume_service.py`, `LINKEDIN_SYSTEM_PROMPT`) — added a "GROUNDED IN THE RESUME" block (source-anchored naming + light anti-climb + no keyword-stuffing + authentic achievement narrative). It had ZERO anti-fabrication instruction before. Research: pure keyword optimization underperforms authentic writing, so this is honest AND more effective.
- **Executive template ATS fix** (`pdf_service.py`) — moved contact info OUT of the canvas header band (which ATS parsers skip) INTO the body text flow. Smoke test confirms email/phone now extract from the body. `_exec_draw_header` no longer draws contact; `_build_executive` prepends a centered contact line.
- **Score-label honesty** (`OptimizeTab.tsx`) — "✓ ATS Cleared" + "Keywords were already strong" now gated on `strongScore = optimizedScore >= 75`. Below 75 shows "Moderate match" + honest "your keyword match is moderate — room to improve" copy. (62 no longer reads as "cleared".)
- **Model pinning — NOT hardcoded (deliberate).** Optimizer default is still the `claude-sonnet-4-6` alias; pinning to a dated snapshot must be done via the `CLAUDE_MODEL` env on Render using the exact snapshot id from the Anthropic console (a wrong string breaks every optimize call, so not guessed in code). ⚠️ ACTION: set `CLAUDE_MODEL` on Render to the dated Sonnet snapshot. Scorer + Haiku calls are already pinned (`claude-haiku-4-5-20251001`).

### ✅ Optimizer patches — title promotion + qualifier-stripping (session 21). `py_compile` clean. From live testing (Ricardo AE, Devon SQL, Marcus Excel).
Multi-industry testing (teacher/data-analyst/sales) confirmed all hard named tools/certs/methods excluded 100%, but surfaced two repeatable residual climbs, now patched in `OPTIMIZER_SYSTEM_PROMPT`:
- **Title promotion:** Ricardo's headline was changed "Sales Representative" → "Account Executive" (the target role). Strengthened TITLE INTEGRITY to explicitly cover the name-headline + summary title and forbid promoting to the target role's title.
- **Qualifier-stripping / feature-elaboration:** Devon's "SQL (basic queries)" → "SQL"; "simple dashboards" → "dashboard development"; Marcus's "Excel" → "Excel (pivot tables, VLOOKUP)". New "PRESERVE SELF-LIMITING QUALIFIERS" rule in the SAME-LEVEL section: keep "basic/simple/assisted with"; don't add sub-features to a named tool not in the original.
- **Validate after deploy:** re-run Ricardo → header stays "Sales Representative"; re-run Devon → "SQL (basic queries)" keeps "basic", no "business intelligence" climb; re-run Marcus → Excel stays plain.

### ✅ Dead-code audit + deletion (session 21). Verified: `tsc -b` 0 errors + `vite build` green; backend `py_compile` + AST + dangling-ref sweep clean. Research-backed (YAGNI/DevIQ; zombie-API security — hackread/Microsoft Defender; unused-dep supply-chain risk — CISA Sept-2025 npm compromise).
Two-pass audit found no legacy code corrupting the primary (semantic) scan/optimize path — only dead code from two removed features + streaming-migration leftovers. **Deleted:**
- **Frontend:** `features/resume-templates/` (whole dir — dead `@react-pdf` client PDF system, superseded by backend ReportLab), `features/workspace/ToolsTab.tsx`, `features/workspace/SkeletonLoader.tsx`, `utils/scoreUtils.ts` (dead duplicate of OptimizeTab's local `scoreToPercentile`). Removed 4 dead API client fns from `resumeApi.ts` (`generateCoverLetter`, `optimizeLinkedIn`, `generateProfessionalSummary`, `enhanceBullet`). Dropped `@react-pdf/renderer` from `package.json` (was already tree-shaken out of the bundle — removal is a supply-chain/install-size win, NOT a bundle-size win; corrected an earlier wrong claim).
- **Backend:** removed 4 zombie endpoints (`/cover-letter/generate`, `/linkedin/optimize`, `/api/tools/professional-summary`, `/api/tools/enhance-bullet`) + their route-table entries + imports; removed 5 now-dead fns from `resume_service.py` (`generate_cover_letter`, `generate_linkedin_optimization`, `generate_professional_summary`, `enhance_bullet_point`, `_parse_linkedin_text`) + their `SUMMARY_SYSTEM_PROMPT`/`BULLET_SYSTEM_PROMPT` constants; trimmed `models/tools_models.py` to just `ResumeDocxRequest` (the live one — kept the file).
- **Kept (verified live or defensive):** rule-based fallback scorer + keyword extraction (no-JD / semantic-failure safety net), legacy stemmer fallback, HS256 auth path (defensive — verify before removing), duplicate `ACTION_LINE_STARTERS` in ats_service + resume_parser (both live; optional dedupe later).
- **Memory-drift correction:** the session-13 note claiming `scoreUtils` + `SkeletonLoader` were "removed" was inaccurate — both were still present until this pass (now actually deleted).
- The app streams cover/LinkedIn/interview, so the non-streaming twins were the dead ones; the streaming endpoints + helpers remain untouched.

### ✅ Security hardening (session 21). `py_compile` clean. Research-backed (OWASP JWT algorithm-confusion; Intigriti/Offensive360 CORS). Full audit found NO critical/high live vulns — these are defense-in-depth.
Audit confirmed strong posture: Stripe webhook signature-verified, no secrets in code or exposed to client (all `VITE_` vars public-safe), no PII in logs, no XSS sinks, comprehensive security headers (CSP/HSTS/X-Frame-Options), HTTPS redirect, JWT sig+expiry verified, rate limiting, RLS-scoped data (JWT user_id, no IDOR), CSRF N/A (bearer auth, `allow_credentials=False`). Hardening shipped:
- **JWT: HS256 legacy path removed** (`supabase_service.py` `verify_token`) — now accepts **RS256/ES256 only** via JWKS + verifies `audience="authenticated"`. Retires the dead-but-trusted HS256 branch (Supabase moved to asymmetric keys); restricting to one algorithm family is OWASP guidance against algorithm-confusion. `SUPABASE_JWT_SECRET` no longer referenced in code. ⚠️ ACTION: remove `SUPABASE_JWT_SECRET` from Render env (no longer used).
- **CORS tightened** (`main.py`) — removed the broad `allow_origin_regex=https://ai-resume-platform[...]\.vercel\.app` (attacker could register `ai-resume-platform-evil.vercel.app` to match it). Now exact-origin allow-list only (`ALLOWED_ORIGINS` env + defaults). Low real risk given `allow_credentials=False`, but defense-in-depth. ⚠️ NOTE: Vercel *preview* deploys can no longer call the API unless added to `ALLOWED_ORIGINS`; production alias unaffected.
- **Dependency advisories** (npm 8 / 3 "high") are all **build/dev tooling** (Vite dev-server, PostCSS, picomatch, flatted, js-yaml) — not shipped to the production static bundle. Run `npm audit fix` (non-breaking) + `pip-audit` as routine hygiene; pin `PyJWT[crypto]` exactly. (Not yet done.)

### ✅ Fabrication fix — "campaign performance analysis" case (session 21). `py_compile` clean + detector unit-tested; `tsc -b` 0 errors. (Vite bundle not run in-sandbox — macOS rollup binaries in mounted node_modules can't load on Linux; confirm with local `npm run build` / Vercel.) From a career-changer test (Nadia, teacher→data analyst): the optimizer aced every hard trap (pulled SQL/Tableau/Python from Projects, computed ~9mo relevant vs 8yr teaching, withheld CTEs/window functions) but then **fabricated a whole skill off the missing-keywords list** — added "campaign performance analysis" (which the scorer itself flagged as a gap) to the Skills section. 72→72, so it took fabrication risk for zero score gain. Research-backed (extractive-faithful rewriting: hakunamatata/swiftscout; entity-diff hallucination detection: arXiv 2509.03531).
- **Prompt (`resume_service.py`):** rewrote KEYWORD INTEGRATION — the Missing-keywords list is now framed as a **DANGER list, not a to-do list**: a missing term may appear in output ONLY if a specific existing bullet already demonstrates it; never added to Skills/summary/bullet as a standalone claim; "campaign performance analysis" used as the worked example. Softened ATS SCORE CONTEXT so **honesty outranks score** (a small honest gain / flat score is the correct result when the candidate lacks the requirement).
- **Deterministic skill-diff safety net (`resume_service.py` `detect_unsupported_skills`, Layer 1 of the checker spec):** pure-code, extracts the optimized Skills section, flags any skill whose HEAD term is absent from the original resume (drops parentheticals so "SQL (joins, subqueries)" isn't split; generic-word stoplist avoids flagging reframes). Unit test: flags "campaign performance analysis" + "Google Analytics", NOT "subqueries" or reworded reals. Wired to `main.py` result payload as `unverifiedSkills`; `OptimizeTab` shows a red **"Double-check these before you use this resume"** callout (human-confirm, never auto-removed).
- **QA fixes:** bullet-count no longer inflated (`bulletChanges` capped at original bullet count, dropped the +added-bullets fudge that over-counted on reformatting); recruiter_verdict trailing/ wrapping quotes stripped (`semantic_ats_service`); **ungrounded "Top X% of applicants" percentile replaced with honest match-strength labels** (Strong/Good/Moderate/Fair/Weak match) since there's no applicant-pool data to claim a percentile.
- **Validate after deploy:** re-run Nadia → "campaign performance analysis" no longer in the optimized Skills (prompt), and if any fabrication slips through it shows in the red "double-check" callout (detector). Score labels read "Moderate match" not "Top 40%".

### 🔲 P2 — at ~50 users — do NOT start pricing/free-tier changes until P0 analytics have run against a real cohort
1. Referral program ("refer → both get a month of Pro").
2. Demo / sample-scan on the landing page (no signup required to see output).
3. Trial-or-money-back decision, **informed by analytics** (don't reflexively cap free scans — 2026 data shows freemium often yields more total customers per visitor).
4. Tracker upgrade tease.
5. Changelog / "What's new".
6. Render starter plan ($7/mo, removes 512MB OOM risk) + Supabase PITR ($25/mo, backups).

### 🔲 Next session
1. **Message 20 personal contacts** — offer free resume scans (first users / beta feedback) — platform is fully live
2. **Incorporate via Stripe Atlas** + set up Wise Business
3. **WiPay integration** + launch to Caribbean diaspora
4. **Book meeting** with Caribbean recruitment agency

## Stripe Atlas Notes
- **Cost**: $500 USD (incorporation + first year registered agent), $100/year after
- **Timeline**: 1–2 business days for Delaware incorporation; EIN takes 15–25 business days for non-US founders (no SSN)
- **Non-US founder path**: No SSN required — apply for ITIN after incorporating. Can accept Stripe payments + open Mercury bank account BEFORE EIN arrives
- **Included**: EIN, 83(b) election, Mercury/Brex bank access, $2,500 Stripe credits (good for ~$80k in processing fees at 2.9% + 30¢), ~$50k partner perks
- **Start**: https://dashboard.stripe.com/register/atlas
- **Status**: Not yet started — decision pending

### Deferred (explicit)
- Zod runtime validation for API responses — post-launch hardening
- Resume score history — now scoped as P1 (see session 19 audit)
- JD URL scraper
- Multiple resume versions
- Job match scoring
- LinkedIn import
- International CV format support (UK/EU conventions)
- **Employer-side matching — "authenticity check", lightest version, LATER (Krystel's idea, session 21):** flip the resume↔JD engine to serve companies (post a vacancy → rank/filter applicants). Parked, NOT now. Why: (1) employer-side AI screening is incumbent-owned — Greenhouse/Workday/Lever all shipped it (~79% of F500 applicants already run through AI-ranking ATS); (2) a tool that filters applicants = a regulated AEDT (NYC Local Law 144 annual bias audits + public impact ratios, EU AI Act "high-risk", EEOC four-fifths adverse-impact liability) — too heavy/legally risky for a solo non-technical founder; (3) it flips the customer B2C→B2B and contradicts the just-shipped candidate-ally brand. The kernel worth keeping: the engine + anti-fabrication verifier is dual-use — if the candidate side gets traction, the lightest employer play is an "is this resume padded/real" authenticity check for SMBs without an ATS (plays to our honesty asset, doesn't fight Workday). Revisit only post-validation.
- **Interview-prep output distinctiveness (from Khareena's "duplicate output" question, session 21):** interview prep runs at `temperature=0` (deterministic), so two users pasting the *same* resume+JD get the *same* questions. Not a privacy issue (no cross-user leakage — verified: per-request generation, no response cache, Anthropic prompt caching has "no effect on output token generation" + org/workspace isolation + ZDR; resume/cover/LinkedIn already stochastic at the API default temp 1.0). It's a *quality* lever: if users report interview questions feeling generic/templated, nudge interview-prep temperature to ~0.5–0.7 for variation, trading some run-to-run consistency. **Wait for the signal — don't change preemptively.** File: `resume_service.py` `stream_interview_prep` (temp=0 at ~L977).

*(Removed from deferred — already implemented: API v1 versioning, DOCX download, Interview Prep.)*

### Platform Self-Learning (post-launch, ~200 users)
Industry standard in 2026 for Claude-backed SaaS is **retrieval-augmented prompting** (dynamic few-shot from own database) — NOT fine-tuning (Anthropic doesn't offer it for Sonnet/Haiku).

**Three things to build in order of impact:**

1. **Dynamic few-shot injection** (highest value) — When a user submits for optimization, query `scan_results` for the 1–2 previous jobs with the highest `score_improvement` in a similar role/industry, prepend as examples in the optimizer prompt. Claude sees real before/after pairs from our own users. Prerequisite: 200+ completed optimizations in DB; below that threshold, examples can *hurt* (documented "few-shot collapse" phenomenon). Implementation: ~1 Supabase query + ~20 lines in `resume_service.py`.

2. **Role/industry keyword intelligence** — Aggregate `missing_keywords` across scans grouped by job title. Build a dynamic "users who included X, Y, Z scored 15 points higher for PM roles" dataset injected into optimizer prompt. Data already exists in `scan_results`.

3. **Prompt A/B testing** — Log prompt version per result, track mean score improvement per variant, run experiments. Tools: Statsig or Arize.

**Action required at 200 users:**
- Add `high_performer boolean` column to `scan_results` (score_improvement ≥ 15)
- Build retrieval query in `resume_service.py`
- A/B test with vs. without injected examples

## Dev Commands
```bash
# Backend (from ai-resume-platform/backend/)
source .venv/bin/activate && uvicorn main:app --reload --port 8000

# Frontend (from ai-resume-platform/frontend-app/)
npm run dev

# Stripe webhook listener (separate terminal, keep running)
stripe listen --forward-to localhost:8000/api/payments/webhook
```

## Key Files
| File | Purpose |
|------|---------|
| `backend/main.py` | FastAPI router — all endpoints incl. Stripe webhook |
| `backend/services/semantic_ats_service.py` | Claude-powered ATS scorer (6 dimensions) |
| `backend/services/resume_service.py` | Optimize, cover letter, LinkedIn via Claude |
| `backend/services/supabase_service.py` | Supabase client — get/set pro status, user lookup |
| `backend/services/ats_service.py` | Rule-based scorer (keyword extraction fallback) |
| `backend/services/resume_parser.py` | Section/contact parser |
| `backend/models/` | 4 Pydantic request models (scan, optimize, cover_letter, linkedin) |
| `frontend-app/src/pages/WorkspacePage.tsx` | Thin shell — imports from features/workspace/ |
| `frontend-app/src/features/workspace/` | ScanTab, OptimizeTab, CoverLetterTab, LinkedInTab, shared |
| `frontend-app/src/components/ErrorBoundary.tsx` | Per-tab error boundary with "Try again" button |
| `backend/services/exceptions.py` | AIUnavailableError — propagates Claude downtime as 503 |
| `frontend-app/src/pages/LoginPage.tsx` | Login page |
| `frontend-app/src/pages/SignupPage.tsx` | Signup page with email confirmation |
| `frontend-app/src/app/AuthContext.tsx` | Supabase auth context — user, session, signIn, signUp, signOut |
| `frontend-app/src/services/supabase.ts` | Supabase client (uses VITE_SUPABASE_URL + VITE_SUPABASE_ANON_KEY) |
| `frontend-app/src/api/resumeApi.ts` | All API calls, axios config |
| `frontend-app/src/app/AppShell.tsx` | Router — all routes incl. /login, /signup, /privacy, /terms |
| `frontend-app/src/app/ThemeContext.tsx` | Dark/light theme context with localStorage + OS fallback |
| `frontend-app/src/features/resume-templates/` | 3 template configs + ResumePDF renderer |
| `frontend-app/src/styles/globals.css` | CSS primitives + semantic tokens; GitHub dark palette |
| `backend/services/pdf_service.py` | ReportLab PDF generation for all 3 resume templates + cover letter |
| `frontend-app/src/pages/PrivacyPage.tsx` | /privacy route |
| `frontend-app/src/pages/TermsPage.tsx` | /terms route |

## Project Rating
**Current: 9.5/10** — fully deployed and working end-to-end. Auth (email + Google), pro-status, Stripe checkout + webhook + billing portal, PDF/DOCX downloads, all AI features confirmed live. Supabase RLS + grants fixed. Analytics (PostHog) + error monitoring (Sentry) live. Session persistence, feedback capture, GDPR deletion, retries shipped. Zero known production bugs.
Gap to 10: post-launch Zod validation (deferred by design); reliability hardening (Render starter plan, Supabase PITR) not yet done.

## Path to 100% — completeness by lens (session 20 assessment)
Single-number completeness is misleading; assess by lens. IMPORTANT framing: only the first lens has a true finish line. Chasing 100% on all four before getting real users is sophisticated procrastination — lenses 3 and 4 are *locked* until a cohort exists, and lens 2's big decisions should be made *from* that data.

### 1. Launch-ready v1 — ~90% → 100% (finite; finish it, it's cheap)
- Activate back-burner items: Resend welcome email + Stripe Customer Portal.
- Verify untested prod paths: account deletion (auth-user step w/ `sb_secret_*`), billing portal round-trip, Google on a fresh account.
- Reliability: Render starter plan ($7/mo — 512MB free tier OOMs on spikes) + Supabase PITR ($25/mo backups). This is the real reason it's not "100% production-grade."
- CI gate: run pytest + `tsc` on every push; add tests on money paths (scan, optimize, Stripe webhook).
- Alert on `/health` (deps), not just `/ping` (liveness).

### 2. Growth / business engine — ~45% → 100% (never fully terminal; = four measured loops running)
- **Acquisition:** no-signup demo/sample scan on landing + SEO surface (programmatic "ATS checker for {role}" pages, a few articles). Zero organic today — cheapest channel for a bootstrapped tool.
- **Activation:** first-run guidance so signup→first-scan is high.
- **Monetization:** trial-vs-guarantee decision *from data*; pricing/free-tier tuned *from data*; referral loop.
- **Retention:** full lifecycle email sequence automated (welcome + 48h + 7-day + post-purchase) via a scheduler.
- 100% = each loop instrumented, measured, improving — not just "built."

### 3. Mature self-improving product — ~30% → 100% (gated on ~200 optimizations)
- The three builds in the Platform Self-Learning section: dynamic few-shot injection, role/industry keyword intelligence, prompt A/B testing.
- Add an eval harness to *prove* scan/optimize output quality is good and improving.
- Product depth (deferred list): multiple resume versions, JD URL scraper, job-match scoring, LinkedIn import, international CV formats.
- Feedback capture (shipped) is the correct first step; the intelligence layer is future work.

### 4. Evidence / validation — ~0% → 100% (the real one; more building does NOT help)
- A real cohort with defensible numbers: activation (signup→scan), scan→optimize, free→paid, N-day return, and the self-reported response/interview rate from the feedback card.
- A handful of user conversations.
- 100% = data-backed confidence in PMF (activates, converts at a defensible rate, retains, reports outcomes).

### The only sequencing that works
1. Finish the cheap last 10% of lens 1 (reliability + activation) — a few days.
2. Message the 20 contacts; watch funnel + feedback for 2–4 weeks.
3. Let that data decide: sharpen output quality (if they don't return) vs. build growth loops (if they do).
The highest-value next output is NOT more code — it's the first real evidence of whether this converts.

---
> Source: [AkilHarrington/ai-resume-platform](https://github.com/AkilHarrington/ai-resume-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
