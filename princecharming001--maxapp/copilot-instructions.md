## maxapp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo shape

Monorepo for **Max**, a looksmaxxing coaching app.

- `backend/` — FastAPI API (Python **3.12**), the main server. Postgres via Supabase. (The root `README.md` is stale: it says "FastAPI + MongoDB / Python 3.8" — neither is true.)
- `mobile/` — Expo / React Native app (Expo SDK 54, RN 0.81), iOS-first, ships to TestFlight via EAS.
- `cannon_facial_analysis/` — a **separate** FastAPI + MediaPipe service computing face-scan metrics. The backend calls it over HTTP (`settings.facial_analysis_api_url`); it is not imported.
- `web/` — a few static/Stripe-embedded pages. `docs/`, `legal/`, `data/`, `rds_templates/` — assets/docs.

## Commands

### Backend
```bash
cd backend
# CRITICAL: Python 3.14 breaks langchain/pydantic at import. Use 3.12.
# A prebuilt venv exists at backend/.venv312 (create with: python3.12 -m venv .venv312 && .venv312/bin/pip install -r requirements.txt)
.venv312/bin/python -m uvicorn main:app --host 0.0.0.0 --port 8000   # add --reload for dev
.venv312/bin/pytest                                                  # all tests
.venv312/bin/pytest tests/test_chat_routing.py::test_name -q         # one test
```
- The local backend connects to the **production Supabase DB** — local test users/data land in prod.
- For reliable local chat, pin the LLM: prefix uvicorn with `LLM_PROVIDER=openai` (the default `gemini` has no local key and the failover hits a Claude timeout, returning the "trouble reaching my brain" fallback).
- Prod start (Render, see `backend/Dockerfile`): `uvicorn main:app --host 0.0.0.0 --port ${PORT}`.

### Mobile
```bash
cd mobile
npx expo start --lan          # or: npm run start:clear
# compile-check one file without a full bundle:
node -e "require('@babel/core').transformFileSync('screens/x.tsx', {presets:['babel-preset-expo']})"
```
- No JS unit-test runner. UI tests are **Maestro** flows in `mobile/maestro/*.yaml`.
- iOS builds: `eas build --platform ios --profile production --auto-submit`. `buildNumber` lives in `app.json`. The production API URL for **builds** comes from `eas.json` (builds ignore the `.env*` files).
- **Local dev API override lives in `mobile/.env.development.local`** (gitignored; loaded ONLY when `NODE_ENV=development`, i.e. `expo start`) — point it at the Mac's **LAN IP** for a real device, or `localhost` for the simulator. **NEVER use `mobile/.env.local`** for this: Expo loads that in *every* environment including production exports, so it leaks into OTA bundles and points every phone at localhost (see the OTA notes under Deploy & ops).

## Backend architecture

- **Entry** `backend/main.py` registers ~23 routers under `/api` (auth, users, scans, payments, courses, chat, schedules, maxes, marketplace, personalization, achievements, …). A global exception handler redacts internals in production and maps DB-connectivity errors to a clean 503.
- **DB** `backend/db/sqlalchemy.py` — SQLAlchemy async + asyncpg against Supabase. Production **must** use the transaction pooler: host `aws-…pooler.supabase.com`, `SUPABASE_DB_PORT=6543`, user `postgres.<project-ref>`. **Gotcha:** never send `search_path` as an asyncpg startup parameter through the pooler — Supavisor closes the connection on the first query ("connection was closed in the middle of operation"). The DB role already defaults `search_path` to `public, extensions`. `/health` returns `{build, db}`; boot logs print `[DB] mode=transaction|session|direct`.
- **Config** `backend/config.py` — pydantic settings, all from env. `is_production` hard-gates dev-only endpoints (faux-signup, dev Google, test-activate) so they can't mint paid accounts in prod.
- **LLM** `backend/services/lc_providers.py` — multi-provider with failover, selected by `LLM_PROVIDER` (`huggingface` fine-tuned default, `gemini`, `openai`, `mistral`). `backend/services/claude_service.py` is Anthropic single-shot (used for task guides). No provider SDKs are imported outside these two files.
- **Chat agent** `backend/services/lc_agent.py` — a LangChain tool-calling `AgentExecutor` (~22 tools: schedule CRUD, `recommend_product`, `search_knowledge`, `web_search`, `remember_about_user`). `build_agent_system_prompt` assembles the prompt: persona (`services/prompt_constants.MAX_CHAT_SYSTEM_PROMPT`) + appended voice/MCQ/product-link rules + injected `KNOWN PROFILE` (user facts). `backend/api/chat.py` routes a turn through a **fast RAG path** (`answer_from_rag`, early-returns) OR the **full agent** (`run_chat_agent`) — they are mutually exclusive. MCQ markers `[CHOICES]a|b|c[/CHOICES]` / `[CHOICES_MULTI]` are emitted by the model and parsed out into a `choices` array in `api/chat.py`.
- **RAG / maxxes** the five programs are `skinmax`, `hairmax`, `fitmax`, `heightmax`, `bonemax`; their coaching docs live in `backend/rag_content/<maxx>/`.
- **Personalization** `services/personalization.py` (unified brief) + `services/user_facts_service.py` (the persisted `KNOWN PROFILE` blob) — injected into the chat prompt; this is the "never re-ask known facts" mechanism.
- **Schedules** `services/master_schedule.py`, `schedule_master_merge.py`, `schedule_*.py` build/merge a user's program schedule. **Task guides** (`services/task_guide_service.py`) are LLM-generated step-by-step guides, cached in the `task_guides` table and pre-warmed via `pregenerate_for_schedule`.
- **Marketplace** `backend/api/marketplace.py` + `backend/data/product_catalog.yaml`. Native maxes are **included** in the subscription (Chad/premium = 3 active-program slots, 7-day swap lock; legacy Lite = 2, grandfathered); creator courses are paid via **Apple IAP** (`com.cannon.creator.*`). `product_catalog.yaml` is the **only** source for product recommendations — `services/link_validator.py` rejects any URL not in it.
- **Payments** **Apple IAP (StoreKit) ONLY** — `services/apple_iap_service.py` verifies via the App Store Server API. **Stripe billing is retired** (every Stripe subscription endpoint 410s via `STRIPE_BILLING_RETIRED`; webhook/`/status` stay mounted inert). One plan: **Chad** (`premium`); Chad Lite (`basic`) retired but grandfathered, `basic→premium` everywhere. **Trial = subscription = full access** (never gate on trial-vs-sub). See `MAX_BIBLE.md` for the canonical product + entitlement truth.

## Mobile architecture

- **Entry** `mobile/App.tsx` → `navigation/RootNavigator.tsx`. The root `Stack.Navigator` is **keyed on auth/paid state** (`stackKey`) — flipping `isPaid` remounts the whole navigator, swapping the unpaid funnel for the paid app. Post-purchase routing depends on this remount.
- **API** `mobile/services/api.ts` — axios client. `resolveApiBaseUrl()` chooses the base URL: production from `.env`, or in dev swaps a loopback URL to the Metro LAN host (falling back to prod if it can't detect one). Auth via Bearer token in `context/AuthContext.tsx`.
- **State** `@tanstack/react-query`. Use the canonical keys in `lib/queryClient` (`queryKeys`) — e.g. all `/active/full` fetches must use `queryKeys.schedulesActiveFull` or achievement celebrations are routed to the wrong cache and lost. Task toggles use optimistic updates (no network-gated spinner).
- **Auth tiers** `isPaid` = any subscription; `isPremium` = Chad (premium) or admin. Gate Chad-only features (e.g. daily face rating) on `isPremium`, not `isPaid`.
- **DevDrawer** `components/DevDrawer.tsx` switches guest/onboarding/paid by calling faux-signup endpoints, which are **404 in production** — it only works against a local backend (see `.env.local`).
- **Feature flags** `constants/featureFlags.ts` (`faceScan`, `onboardingV2`, `todayV2`, …) gate funnel and home behavior.

## Deploy & ops

- **Backend → Render** (service `maxapp-api`), auto-deploys on push to `main`. Render env vars are the prod config source of truth (DB pooler creds, `GOOGLE_IOS_CLIENT_ID`, Stripe, Apple keys). A **failed** deploy keeps serving the last good build — confirm a deploy actually went live via `/health`'s `build` string and `[DB] mode=transaction` in the boot logs.
- **Mobile → EAS → TestFlight** (bundle id `com.cannon.mobile`, App Store Connect app id `6761345332`). Native config (Google sign-in URL scheme, IAP, etc.) is baked into the binary, so changes there require a new build.
- **OTA (`eas update`) env — structurally safe since 2026-07-15.** `eas build` uses `eas.json`'s `build.<profile>.env`; **`eas update` runs `expo export` locally and loads the `.env*` files instead.** The local-dev API override therefore lives in **`mobile/.env.development.local`** (loaded ONLY when `NODE_ENV=development`, i.e. `expo start`), never in `.env.local` (which Expo loads in EVERY environment, including production exports). `mobile/.env` holds the production defaults, so an OTA always bakes the real Render URL.
  - **ALWAYS publish with `--clear-cache`:** `cd mobile && npm run ota -- --message "..."` (the script includes it). Metro CACHES the transform that inlines `EXPO_PUBLIC_*`, so an OTA can ship a stale bundle with the OLD inlined API URL **even when the env is correct and the logs say `env: load .env`**. On 2026-07-15 a no-cache-clear republish shipped the SAME poisoned bundle twice (identical launchAsset hash) and the app stayed broken.
  - **Verify before trusting an OTA:** `npx expo export --platform ios --clear --output-dir /tmp/x` then `strings /tmp/x/_expo/static/js/ios/*.hbc | grep -c onrender` (must be ≥1) and `| grep -c localhost` (must be 0).
  - **Never reintroduce `mobile/.env.local`** for the API URL — that's what shipped a localhost bundle to every phone on 2026-07-15 ("network error", app unusable). Sanity check any OTA's output: it should log `env: load .env`, never `.env.local`.

---
> Source: [princecharming001/maxapp](https://github.com/princecharming001/maxapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
