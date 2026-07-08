## 10-amenities-module

> Member Amenities module specifics — apply when touching amenities_* tables, supabase/functions/amenities-*, or amenities/attendant/leaderboard/post-health/floor-integrity frontend.


# Member Amenities module — implementation conventions

Source of truth: `Member Amenities System - PRD.md` Part B (B.1–B.8) and the Codebase Alignment Review. Where the conceptual sections (§7.2–§7.3) differ from Part B, **Part B wins**.

## Architecture spine
- **Mirror the member list; never live-call FlexWash per activation.** Every activation resolves against the server-side mirror `amenities_member`. The post never sees the CSV or the API.
- **Phase 1 (now, Margate pilot):** bulk mirror is loaded by **`amenities-import-members`** from the FlexWash "memberships On" CSV export (`id, name, email, phoneNumber, vehicles`). Sample: `data/FlexWash Member Export - import format sample (memberships On).csv`. **Presence in the file = active.** No status/expiry columns → `valid_until = null`.
- **Phase 2 (later):** `amenities-sync-members` pulls the read-only members API (only once FlexWash ships a higher rate limit). Reuses the same parse→normalize→guard→merge core.
- **Merge is key-preserving, keyed on `flexwash_customer_code`** (the stable identity anchor; `phone_e164` is a unique *lookup*, not identity). **Upsert, never delete-and-reinsert.** Never touch `amenities_member_tag` / `amenities_vehicle_tag` rows where `source='rt_os'`. RT OS is the system of record for all preference tags; tags are **not** written back to FlexWash.
- **Fail-safe:** stage first, then refuse the merge and keep the prior mirror if the file/pull is empty, implausibly small (< 50% of `prev_active_count`, a Settings threshold), or error-heavy. Deactivate only by absence in a *successful, sanity-checked* refresh. Log every load to `amenities_member_import`.

## Identity model (distinct on purpose)
- Floor attendants exist **only** as `amenities_attendant` rows (hashed punch code), **never** Supabase-auth users, **never** Connecteam users. "Logged in" == an open `amenities_attendant_session`.
- The leaderboard's "who was working" denominator is `amenities_attendant_session` **alone** — do **not** join `flexwash_employees`.
- Punch-code hashing happens **server-side** via `amenities-attendant-admin`; `punch_code_hash` is never sent to or stored in the browser and is not SELECT-able under RLS.
- Position (First/Second/Third) is **derived at runtime** from punch-in order among open sessions, not stored. Stall ownership comes from `amenities_lot_map` for the current headcount. `max_attendants` is a per-location Setting (default 3).

## Net-new patterns (no codebase precedent — build deliberately, design-review required)
1. **Per-device auth (B.7).** Device holds a per-device bearer key (not the anon key), scoped to `amenities-activate`, `amenities-member-cache`, `amenities-events-sync`, `amenities-heartbeat`, `amenities-help`. Key shown **once** at provisioning; store only `amenities_device.device_key_hash`. **Every** device-facing function checks `amenities_device.revoked` on every call. Shape = **validate-bearer-then-service-role-write** (same as the HMAC webhook). The device speaks raw HTTPS `POST /functions/v1/<fn>` — not `functions.invoke`.
2. **Tablet token & board reads — RESOLVED (June 2026), supersedes PRD §B.5 native Realtime on Lovable Cloud.** The PRD proposed Supabase Realtime on `amenities_service_card` + a Supabase-accepted custom tablet JWT (location-claim RLS keyed on `auth.jwt() ->> 'location_id'`). **This is NOT achievable on Lovable Cloud** (the project JWT secret is neither injected nor readable, custom-JWKS/third-party-issuer registration is not exposed, and floor attendants must never be Supabase-auth users — see `40-supabase-lovable-cloud-migrations.mdc` §12). So **do NOT** `ALTER PUBLICATION supabase_realtime ADD TABLE …` for the tablet and **do NOT** add tablet RLS keyed on a Supabase-verified JWT. The settled architecture:
   - **Our own token, our own verifier.** `amenities-attendant-session` mints an HS256 token with **`AMENITIES_TABLET_JWT_SECRET`** (a value WE control — *not* the project secret; project-prefixed because `SUPABASE_*` is reserved). Mint/verify both live in **`_shared/attendantAuth.ts`** (`mintTabletJwt` / `verifyTabletToken`). Claims: `att_role:'amenities_tablet'`, `location_id`, `session_id`, `attendant_id`; `role`/`aud` are `'authenticated'` but vestigial (Supabase never verifies this token).
   - **Board via session-validated edge function + polling.** `amenities-board` verifies the token → confirms the session is still open / not idle → service-role read scoped to the token's `location_id`; `amenities-serve` is the write. The dashboard uses React Query `refetchInterval` polling (the PRD's "correct on polling alone" path, here the **primary** mechanism). **Reads never extend the session; only attributed writes (serve/welcome/help) bump `last_activity_at`.**
   - **Shared floor logic in `_shared/`.** Punch-in ranking + lot-map ownership (`rankPositions`/`positionLabel`/`buildOwnership`) live in **`_shared/floorMap.ts`** so both `amenities-attendant-session` and `amenities-board` import them — never cross-import one function's `_lib.ts` from another (it fails at deploy; §12).
   - Native Realtime/RLS push is a future upgrade only if the project moves to an external Supabase.

## Edge functions to build (names are the contract — B.4)
`amenities-import-members` (admin-authed), `amenities-activate`, `amenities-member-cache` (NDJSON of per-device salted hashes + `{"end":true,"count":N}` trailer + Content-Length), `amenities-events-sync` (idempotent on `(device_id, event_id)`), `amenities-heartbeat`, `amenities-device-watcher`, `amenities-attendant-session`, `amenities-attendant-admin`, `amenities-welcome-complete`, `amenities-card-sweep`, `amenities-override` (staff-authed), `amenities-help` (device-authed), `amenities-membership-webhook` (HMAC, optional), `amenities-online-signup`, `amenities-trial-grant`, `amenities-sms-inbound` (Twilio sig), `refresh-amenities-performance`, `refresh-amenities-integrity`, `amenities-sync-members` (Phase 2).

## Frontend placement (`src/navigation/config.ts`)
- Attendant Dashboard → `stations` section (punch-code login, not OAuth). Orange new-member treatment must use border + pill/label, **never color alone** (accessibility).
- Member Amenities leaderboard → `leaderboards` section.
- Post Health → `dashboards` (role-gated). Verification Queue + Floor Integrity → `reports` (role-gated). *(Verification Queue moved from `dashboards` to `reports` per Tom, 2026-06-18; route `/reports/amenities-verification`.)* Settings tabs → `settings`.
- Settings tabs register in `src/pages/Settings.tsx` exactly like existing ones, gated `requireRole:["admin"]` + `featureFlag:"member-amenities"`. Mirror `UserManagement.tsx` / `FlexwashEmployeeMappingSettings.tsx` (React-Query table + shadcn `Table`/`Dialog`/`Button` + `sonner`).
- UI mockups are the design reference: `Attendant Tablet - UI Mockup.html`, `Manager - Post Health (UI Mockup).html`, `Manager - Floor Integrity (UI Mockup).html`, `Post LCD Screen - Messages & Mockups.html`.

## Nightly scheduled-job order (Lovable-scheduled — order matters)
The amenities nightly jobs are service-role functions with **data dependencies**, so they must run in this order (Lovable owns the schedule; Cursor can't cron). Each consumes the prior one's output for the ET business day:
1. **`amenities-card-sweep`** — closes the day's open/unserved cards and expires provisional grants, so the served/missed picture is final.
2. **`refresh-amenities-integrity`** (Slice 12) — writes `amenities_integrity_flag` snapshot rows (incl. `review`-tier) for the day. **Must run before the sampler** so flagged-attendant oversampling can see same-night flags.
3. **`amenities-verification-sample`** — Pass A oversamples `review`-tier flagged attendants (reads step 2's flags), Pass B random-samples non-flagged; also ages sampled+pending rows to `expired`. **Must run before the leaderboard refresh** so the week's untrue/reviewed counts are settled.
4. **`refresh-amenities-performance`** — leaderboard snapshot + §9.3 Service-Verified gate (reads the min-n Setting + the week's reviewed/untrue records).

If a new amenities nightly job reads another's output, slot it into this chain explicitly rather than assuming arbitrary order. (Slice 12's `refresh-amenities-integrity` is the one that must land between the sweep and the sampler.)

## Before merging anything, re-read `20-open-holds.mdc`.

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
