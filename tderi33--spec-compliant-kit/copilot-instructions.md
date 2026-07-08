## 20-open-holds

> HARD STOPS. Unresolved decisions Tom must sign off on. Always loaded. The agent may build surrounding structure but must NOT invent these. HOLD 2 is OPEN; HOLD 1 and HOLD 3 are RESOLVED (kept for the record).


# Open holds — do NOT resolve these in code without Tom's explicit sign-off

**HOLD 2 below is still OPEN.** Build the structure around it, but **stop and flag** rather than inventing a value or physical dimension. If a task would require resolving it, leave a clearly-marked `// HOLD:` stub and surface it for Tom. **HOLD 1 and HOLD 3 are RESOLVED** and kept below for the record + decided values.

## HOLD 1 — Leaderboard blended score (§9.3) — ✅ RESOLVED (Tom, 2026-06-19)
The prior ambiguity (weights + tie-break) is now decided and implemented in `AmenitiesLeaderboard.tsx`:
- **Formula:** `score = coverage_pct + min(tags_added, 10) * 0.5`
- **Tie-break order (desc):** blended score → coverage_pct → served_count → welcomes_count → tags_added → name
- **Service-Verified week gate:** attendants with `untrue_flags_week >= 2` are forced to the bottom on **This Week** only (resets Monday ET)

This is now a product decision of record. Further changes to this formula/tie-break still require Tom.

## HOLD 2 — ADA / member accessibility decisions are unrecorded (§7.8)
§7.8 covers the attendant UI only. The member-facing physical interaction is undecided: **keypad / HELP-button mounting height & reach (ADA 308/309), one-hand operation, low-vision masked entry on the 2.8″ display.** These must be recorded **before Margate fabrication.**
- ❌ Do **not** bake physical dimensions, mounting heights, or a final masked-entry UX into firmware, the LCD rendering spec, or the BOM/wiring without Tom's recorded decision.
- ✅ You may leave the relevant constants/config as flagged TODOs.

## HOLD 3 — §8 verification sampling policy — ✅ RESOLVED (Tom, 2026-06-18)
The original hold: 5 clips/location/day means a typical attendant gets 0–2 reviewed services/week, so Service-Verified (≤1 untrue) hinged on a tiny sample. The undecided parts were the **minimum-n** and **whether integrity-flagged attendants get 100% review**. Both are now decided, and all live as per-location `amenities_settings` values (not buried constants):
- **Minimum-n = 1** (`service_verified_min_n`, default 1). Applied in `refresh-amenities-performance`: below min-n reviewed services in the ET week, Service-Verified is **neither** pass nor fail (`service_verified = null`).
- **Flagged-attendant oversampling: ON, `review` tier only, capped 5/flagged-attendant/ET day** (`verification_full_review_flagged` = true, `verification_full_review_tier` = 'review', `verification_full_review_cap` = 5). Applied **additively** in `amenities-verification-sample` (Pass A oversamples flagged up to the cap; Pass B's random per-location sample draws only from non-flagged attendants, so the oversample never consumes the random budget). Inert until Slice 12 populates `amenities_integrity_flag` with `review`-tier rows.
- The §8 random sample size (`verification_sample_size`, default 5) and pending-expiry window (`verification_pending_expiry_days`, default 7) remain Settings values.

These are now product decisions of record — implement against them; they are no longer a hard stop. Any **further** change to the sampling policy still needs Tom.

---
**If you are about to write code that picks a value for HOLD 2 above, STOP and ask Tom instead.** (HOLD 1 and HOLD 3 are resolved — use the recorded values.)

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
