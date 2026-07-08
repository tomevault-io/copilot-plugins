## 00-rt-os-conventions

> Rising Tide OS (spec-compliant-kit) core conventions. Always loaded. Read before writing any code in this repo.


# Rising Tide OS — non-negotiable conventions

You are working in the production `spec-compliant-kit` (Rising Tide OS) repo. Match how it is already built. Do not introduce a parallel stack, a new server, or new patterns where one already exists.

## Stack (as it actually is)
- **Frontend:** React + Vite + TypeScript SPA. shadcn/ui + Tailwind. **TanStack React Query** for all data. Pages under `src/pages`; stations at routes like `/stations/sales`. Routing in `src/App.tsx` via `<ProtectedRoute>`.
- **Backend:** **Supabase (hosted)** — Postgres, **Deno edge functions** in `supabase/functions/*`, Auth. **There is NO separate server and NO REST router.** `config.toml` holds only `project_id`.
- **Server logic = edge functions**, called from the frontend with `supabase.functions.invoke("fn-name")`, plus PostgREST table queries, plus `SECURITY DEFINER` RPCs `GRANT EXECUTE … TO anon` for public callers (the feedback pattern). Edge functions build a service-role client: `createClient(SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY)`.
- Any spec text that lists REST routes (e.g. `POST /amenities/activations`) is **conceptual only** — implement the named edge functions instead.

## Time — hard project rule
**All day boundaries are ET (`America/New_York`), DST-aware.** Use `_shared/etRange.ts`, `_shared/business-date.ts`, `todayEt()`, `etDayRangeUtc()`. **Never** do naive UTC date math. `useDwellSnapshots.ts → etDayUtcRange` is the reference for "today" windows.

## Auth, identity & RBAC
- Identity: Google OAuth via Lovable proxy (`@risingtidecarwash.com`) → `supabase.auth.setSession`. `useAuth.tsx` reads `role` from **`user_roles`**.
- Gate manager/admin pages with `useRoleAccess` + rows in `role_page_access` + `user_location_access`; `user_profiles.default_location_id`. Route guard: `ProtectedRoute` (`requireRole` / `requireAdmin` / `pageKey="nav:/…"`).
- RLS uses `SECURITY DEFINER` helpers `has_role(uid, role)` and `get_user_location(uid)`; policies are "admin = all, manager = own location."
- **`app_role` is a 2-value enum (`'admin','manager'`). Do NOT alter it. There is no `attendant`/`employee` role** — and there must not be one. Floor attendants are **not** Supabase-auth users.

## Locations
**`public.locations` already exists** and carries `flexwash_id`, `connecteam_id`/`connecteam_scheduler_id`, `livereach_id`/`livereach_tunnel_id`, `nvr_deeplink_camera_id` (+ `nvr_measurement_camera_id`, `nvr_device_id`), `timezone`, `is_active`, lat/long. **Never create a `site` table.** Always `location_id UUID REFERENCES public.locations(id)`.

## FlexWash external-call discipline
- Reuse `_shared/feedback.ts`: `getFlexwashToken`, `normalizePhoneToE164`, `logFlexwashCall`, `feedbackCallsUsedToday`, `adminClient`.
- FlexWash limit (confirmed June 2026): **≤50 token-mints/day, 24h token validity, ≤75 requests per token per day.** Discipline: **mint one token, reuse it.**
- The amenities module uses its **own dedicated token** (minted under existing `FLEXWASH_CLIENT_ID/SECRET`, no new creds) and its **own** counter `amenitiesCallsUsedToday()` against a Settings `flexwash_lookup_budget` (**default 60**, under the 75 ceiling) — independent of feedback's 4/day.
- **Phase 1 bulk member load = CSV import, ZERO FlexWash API calls.** Only per-miss live lookups (Layer 2 + ad-hoc) call FlexWash, one request each.

## Reuse, don't rebuild
- LiveReach: `LIVEREACH_API_KEY`, `livereach_nvr_token_cache`, `livereach-patches/_lib.ts`. Clip deeplink = `app.livereach.ai/security/{cameraId}?t=…` (see `buildLiveReachDeepLink()` / `tunnel-eff-video`).
- Performance snapshots: computed table + scheduled refresh fn — model on `tunnel_associate_performance` + `refresh-tunnel-associate-performance` (structure only).
- Service→tag mapping: existing `service_map` pattern.
- PII: `feedback-purge-pii`, `hashClientIp`, E.164 storage, daily-budget guards.

## House rules for the agent
- Plan before you write. Surface the file list and migration/RLS plan for approval before editing.
- Keep diffs small and reviewable; one logical slice per commit/PR.
- Migrations are forward-only SQL files in the existing migrations dir; never edit a shipped migration.
- If a spec section conflicts with the live codebase, **the codebase wins** — say so and stop rather than inventing.

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
